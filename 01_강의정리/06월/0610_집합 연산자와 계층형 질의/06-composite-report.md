# 복합 활용 — 집계·서브쿼리·집합 연산으로 통계 리포트 만들기

> 지금까지 배운 도구들은 따로따로일 때보다 함께 쓸 때 진짜 힘을 발휘합니다. "장르별 매출 + 전체 합계" 같은 실무 리포트를 한 쿼리로 완성해 봅니다.

`복합 쿼리` `소계 행` `리포트` `집계` `서브쿼리` `UNION ALL` `EXISTS` `인라인 뷰`

## 핵심요약

- 실무 리포트는 한 가지 기능이 아니라, *조인 + 집계 + 집합 연산*을 한 쿼리에 엮어 만든다.
- "그룹별 집계 + 맨 아래 전체 합계 행"은 `GROUP BY` 결과에 `UNION ALL`로 합계 행을 붙이는 패턴이다.
- 월별/연도별 소계처럼 *여러 단계의 소계*도 같은 방식으로 쌓을 수 있다.
- `EXISTS`와 `HAVING`을 함께 쓰면 "조건 A를 만족하면서 집계 결과도 일정 이상"인 대상을 뽑을 수 있다.
- 집합 연산자(`EXCEPT`)를 서브쿼리로 감싸면, "한쪽에만 있는 대상"을 다시 조인·정렬할 수 있다.

## 개념별 정리

### 그룹별 집계 + 전체 합계 행 (UNION ALL 소계 패턴)

**1. 정의**
`GROUP BY`로 그룹별 숫자를 구한 결과 *아래에*, 전체를 합산한 "합계 행"을 `UNION ALL`로 한 줄 덧붙이는 방법입니다.

**2. 왜 필요한가?**
엑셀 표 맨 아래의 "합계" 행처럼, 그룹별 수치와 전체 총합을 *한 표에서* 같이 보고 싶을 때가 많습니다.

**3. 예시**

```sql
-- 장르별 트랙 수 + 맨 아래 전체 합계
SELECT g.Name            AS 장르명,
       COUNT(t.TrackId)  AS 트랙수
FROM genres g
LEFT JOIN tracks t ON g.GenreId = t.GenreId
GROUP BY g.Name

UNION ALL

SELECT '■ 전체합계',
       COUNT(*)
FROM tracks

ORDER BY 장르명;
```

**4. 헷갈리기 쉬운 점**
합계 행을 만드는 둘째 쿼리도 *첫째 쿼리와 컬럼 수·타입이 같아야* 합니다. "■ 전체합계" 같은 라벨 문자열로 첫 컬럼을 채워, 어느 행이 합계인지 표시합니다.

**5. 한 줄 정리**
"그룹별 + 전체 합계"는 `GROUP BY` 결과에 `UNION ALL` 한 줄을 더하는 패턴이다.

> 비유: 반별 인원수를 적은 표 맨 아래에 "전체: 120명"을 한 줄 덧붙이는 것과 같습니다.

### 조건(EXISTS) + 집계(HAVING) 조합

**1. 정의**
`EXISTS`로 "어떤 조건을 만족하는 대상"을 먼저 거르고, 동시에 `HAVING`으로 "집계 결과가 일정 이상"인 대상까지 함께 거르는 방식입니다.

**2. 왜 필요한가?**
"앨범이 2개 이상이면서, 총 트랙도 30개 이상인 아티스트"처럼 *두 가지 종류의 조건*을 동시에 걸어야 할 때 유용합니다.

**3. 예시**

```sql
-- 앨범 2개 이상 & 총 트랙 30개 이상인 아티스트
SELECT ar.Name                    AS 아티스트,
       COUNT(DISTINCT al.AlbumId) AS 앨범수,
       COUNT(t.TrackId)           AS 총트랙수
FROM artists ar
JOIN albums al ON ar.ArtistId = al.ArtistId
JOIN tracks t  ON al.AlbumId  = t.AlbumId
WHERE EXISTS (
    SELECT 1
    FROM albums a2
    WHERE a2.ArtistId = ar.ArtistId
    GROUP BY a2.ArtistId
    HAVING COUNT(*) >= 2        -- 앨범 2개 이상
)
GROUP BY ar.ArtistId
HAVING 총트랙수 >= 30           -- 총 트랙 30개 이상
ORDER BY 총트랙수 DESC;
```

**4. 헷갈리기 쉬운 점**
`WHERE EXISTS(...)`는 *집계 전* 개별 행 단계의 조건이고, 바깥의 `HAVING`은 *집계 후* 그룹 단계의 조건입니다. 두 조건이 적용되는 시점이 다릅니다.

**5. 한 줄 정리**
EXISTS는 "그런 대상인지"를, HAVING은 "집계 결과가 충분한지"를 동시에 묻는다.

> 비유: 입사 지원에서 "자격증 보유(EXISTS)"와 "경력 3년 이상(HAVING)"을 동시에 요구하는 것과 같습니다.

### 집합 연산을 서브쿼리로 감싸 다시 활용하기

**1. 정의**
`INTERSECT`·`EXCEPT` 같은 집합 연산의 결과를 *임시 테이블처럼* 서브쿼리로 감싸, 바깥에서 조인하거나 정렬하는 방식입니다.

**2. 왜 필요한가?**
집합 연산은 보통 ID 같은 *키 목록*만 돌려줍니다. 거기에 이름·이메일 등을 붙이려면, 그 결과를 다시 다른 테이블과 조인해야 합니다.

**3. 예시**

```sql
-- 두 해 모두 구매한 고객(교집합)을 구한 뒤, 이름까지 결합
SELECT c.CustomerId, c.FirstName, c.LastName
FROM customers c
INNER JOIN (
    SELECT CustomerId
    FROM invoices
    WHERE strftime('%Y', InvoiceDate) = '2009'
    INTERSECT
    SELECT CustomerId
    FROM invoices
    WHERE strftime('%Y', InvoiceDate) = '2010'
) inv ON inv.CustomerId = c.CustomerId;
```

**4. 헷갈리기 쉬운 점**
집합 연산을 감싼 서브쿼리에는 별칭(`inv`)이 필요하고, 바깥에서는 그 별칭으로 결과를 가리켜 조인합니다.

**5. 한 줄 정리**
집합 연산 결과도 *임시 테이블*이라, 서브쿼리로 감싸면 자유롭게 조인·정렬할 수 있다.

> 비유: 두 명단에서 공통 인물을 추려낸 쪽지(집합 연산 결과)를 들고, 인사 기록부에서 그 사람들의 상세 정보를 찾아 붙이는 것과 같습니다.

## 코드로 보기 — 장르별 매출 통계 리포트 (합계 행 포함)

```sql
SELECT g.Name                                      AS 장르,
       COUNT(DISTINCT ii.InvoiceId)                AS 청구서수,
       SUM(ii.Quantity)                            AS 판매수량,
       ROUND(SUM(ii.UnitPrice * ii.Quantity), 2)   AS 총매출
FROM invoice_items ii
INNER JOIN tracks t ON ii.TrackId = t.TrackId
INNER JOIN genres g ON t.GenreId  = g.GenreId
GROUP BY g.Name

UNION ALL

SELECT '── 전체합계',
       COUNT(DISTINCT InvoiceId),
       SUM(Quantity),
       ROUND(SUM(UnitPrice * Quantity), 2)
FROM invoice_items

ORDER BY 장르;
```

**코드목적**
장르별로 *청구서 수·판매 수량·총매출*을 한 번에 집계하고, 맨 아래에 전체 합계 행까지 붙인 실무형 매출 리포트입니다. 이번 강의의 거의 모든 도구가 한 쿼리에 들어 있습니다.

**해석**
1단계로 청구 항목(`invoice_items`)·트랙·장르 세 테이블을 조인해 "각 판매 줄에 장르 이름"을 붙입니다. 2단계로 `GROUP BY g.Name`으로 장르별로 묶고, 청구서 수는 중복을 뺀 `COUNT(DISTINCT InvoiceId)`, 판매 수량은 `SUM(Quantity)`, 총매출은 `SUM(단가×수량)`으로 계산합니다. 3단계로 `UNION ALL`을 통해 전체 합계 한 줄을 덧붙입니다. 합계 행도 컬럼 수·타입이 같아야 하므로 첫 컬럼에 `── 전체합계` 라벨을 넣습니다.

**실무 연결**
"카테고리별 실적 + 전체 합계"는 거의 모든 매출/운영 대시보드의 기본 형태입니다. 이 한 쿼리만 변형해도 "지역별 매출 + 합계", "월별 가입자 + 합계"처럼 수많은 리포트를 만들 수 있습니다. 스스로 처음부터 작성할 수 있다면, 실무 SQL의 상당 부분을 다룰 수 있는 수준입니다.

## 직접 해보기

1. "월별 매출 + 각 연도의 합계 행"을 만들어 보세요. (인라인 뷰가 아니라 `UNION ALL`로 합계 행을 덧붙이는 방식)
2. 위의 장르별 매출 리포트에서 "총매출이 큰 순"으로 정렬을 바꿔 보세요. (합계 행이 어디로 가는지 관찰)
3. 2009년 구매 고객 중 2010년에 구매하지 않은 고객(차집합)을 구해, 이름과 이메일까지 함께 출력해 보세요.

## 헷갈리기 쉬운 포인트

- **소계 행의 컬럼 맞추기**: 합계 행을 `UNION ALL`로 붙일 때도 컬럼 수·타입이 본 쿼리와 같아야 한다. 라벨 문자열로 첫 컬럼을 채운다.
- **WHERE EXISTS vs HAVING**: 전자는 집계 전 개별 행 조건, 후자는 집계 후 그룹 조건. 시점이 다르다.
- **집합 연산 결과 재활용**: ID 목록만 나오는 집합 연산은, 서브쿼리로 감싸 다른 테이블과 조인해야 이름 등 상세 정보를 붙일 수 있다.

## 연결되는 개념

- 이전에 알면 좋은 글: [집계 함수와 GROUP BY / HAVING](02-aggregate-functions.md), [서브쿼리](03-subquery.md), [집합 연산자](04-set-operators.md)
- 함께 보면 좋은 글: [계층형 질의](05-hierarchical-query.md) — 계층 결과에 집계를 붙이는 것도 같은 발상입니다.
- 더 찾아볼 키워드: `ROLLUP`, `GROUPING SETS`, `윈도우 함수`, `대시보드 쿼리`

## 셀프 체크

- [ ] `GROUP BY` 결과에 `UNION ALL`로 합계 행을 붙이는 패턴을 작성할 수 있다.
- [ ] 합계 행을 붙일 때 컬럼 수·타입을 왜 맞춰야 하는지 안다.
- [ ] `WHERE EXISTS`와 `HAVING`이 적용되는 시점 차이를 설명할 수 있다.
- [ ] 집합 연산 결과를 서브쿼리로 감싸 조인할 수 있다.
- [ ] 여러 도구를 한 쿼리에 엮어 리포트를 구성하는 흐름을 안다.

**복습 질문 및 답변**

- (기본) 그룹별 집계 아래에 "전체 합계 행"을 붙일 때 어떤 연산자를 쓰나요?
  → `UNION ALL`입니다.
- (이해 확인) 합계 행을 만드는 쿼리가 본 쿼리와 반드시 맞춰야 하는 것은 무엇인가요?
  → 컬럼 개수와 데이터 타입입니다.
- (응용) 교집합으로 구한 고객 ID 목록에 고객 이름을 붙이려면 어떻게 해야 하나요?
  → 집합 연산 결과를 서브쿼리로 감싸 고객 테이블과 조인합니다.

## 한 줄 정리

> 실무 리포트는 집계·서브쿼리·집합 연산을 한 쿼리에 엮어 만들며, "그룹별 집계 + `UNION ALL` 합계 행"은 그중 가장 자주 쓰이는 핵심 패턴이다.
