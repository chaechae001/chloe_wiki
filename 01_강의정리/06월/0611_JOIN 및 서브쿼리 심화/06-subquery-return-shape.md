# 서브쿼리 ② 반환 형태 — 단일행 · 다중행 · 다중컬럼

> 서브쿼리가 값을 "하나" 주느냐, "여러 개" 주느냐, "여러 컬럼"을 주느냐에 따라 바깥에서 쓸 수 있는 연산자가 달라집니다. 형태를 잘못 보면 바로 에러가 납니다.

`서브쿼리` `단일행` `다중행` `다중컬럼` `IN` `EXISTS` `ALL` `ANY`

## 핵심요약

- 서브쿼리는 **반환되는 결과의 모양**으로도 나뉜다: 단일 행 / 다중 행 / 다중 컬럼.
- **단일 행 서브쿼리**: 결과가 1행 1컬럼 → `=`, `>`, `<`, `>=`, `<=` 같은 단일 행 연산자와 함께 쓴다.
- **다중 행 서브쿼리**: 결과가 여러 행 → `IN`, `EXISTS`, `ALL`, `ANY`와 함께 쓴다.
- **다중 컬럼 서브쿼리**: 결과가 여러 컬럼 → `(컬럼1, 컬럼2) IN (서브쿼리)` 형태로 동시에 비교.
- 단일 행 연산자에 여러 행이 들어오면 에러 — 형태와 연산자를 반드시 맞춰야 한다.
- SQLite는 `ALL`/`ANY`를 지원하지 않아, 각각 `MAX`/`MIN` 서브쿼리로 대체한다.

## 개념별 정리

### 단일 행 서브쿼리

**1. 정의**
결과가 **딱 한 행, 한 컬럼**(값 하나)인 서브쿼리다. 그래서 `=`, `>`, `<` 같은 "값 대 값" 비교 연산자와 쓴다.

**2. 왜 필요한가?**
"전체 평균보다 큰", "가장 짧은 트랙 이하인"처럼 **하나의 기준값**과 비교할 때 쓴다.

**3. 예시 (평균 총액보다 큰 인보이스)**
```sql
SELECT InvoiceId, CustomerId, Total
FROM invoices
WHERE Total > (
    SELECT AVG(Total)
    FROM invoices
)
ORDER BY Total DESC
LIMIT 10;
```

**4. 헷갈리기 쉬운 점**
서브쿼리가 실수로 **여러 행**을 반환하면 `=`나 `>` 같은 단일 행 연산자에서 에러가 난다(또는 예측 불가). `AVG`, `MAX`, `MIN`, `COUNT` 같은 집계나 `WHERE Name = '특정값'`처럼 반드시 1행만 나오는 조건을 써야 안전하다.

**5. 한 줄 정리**
단일 행 서브쿼리는 값 하나를 만들어 단일 행 연산자와 비교하는 형태다.

### 다중 행 서브쿼리

**1. 정의**
결과가 **여러 행**일 수 있는 서브쿼리다. `IN`, `EXISTS`, `ALL`, `ANY` 같은 다중 행 연산자와 함께 쓴다.

**2. 왜 필요한가?**
"이 ID 목록 중 하나에 해당하는", "이런 행이 존재하는" 같이 **여러 후보값**과 견줘야 할 때 필요하다.

**3. 예시 (Rock/Jazz 장르 트랙)**
```sql
SELECT Name, GenreId
FROM tracks
WHERE GenreId IN (
    SELECT GenreId
    FROM genres
    WHERE Name = 'Rock' OR Name = 'Jazz'
)
ORDER BY GenreId, Name
LIMIT 10;
```

각 연산자의 의미는 다음과 같다.

| 연산자 | 의미 |
| --- | --- |
| `IN` | 서브쿼리 결과 값들 중 **하나와 일치**하면 참 |
| `EXISTS` | 서브쿼리 결과가 **존재하는지**만 확인 |
| `ALL` | 결과의 **모든 값**에 대해 조건을 만족해야 참 |
| `ANY` | 결과의 값들 중 **하나 이상**이 조건을 만족하면 참 |

**4. 헷갈리기 쉬운 점**
`>= ALL`은 "그 그룹의 **최댓값**보다 크거나 같다"와 같고, `>= ANY`는 "그 그룹의 **최솟값**보다 크거나 같다"와 같다. 이 동치 관계가 SQLite 대체법의 핵심이다(아래 코드 참고). 또 `NOT IN`은 목록에 NULL이 섞이면 예상과 다르게 비는 경우가 있어, 그럴 땐 `NOT EXISTS`가 안전하다.

**5. 한 줄 정리**
다중 행 서브쿼리는 여러 후보값을 IN·EXISTS·ALL·ANY로 견주는 형태다.

> 비유: 출입 명단(여러 명)과 대조해 통과 여부를 정하는 것 — `IN`은 명단에 있으면 통과.

### 다중 컬럼 서브쿼리

**1. 정의**
서브쿼리가 **여러 컬럼**을 반환하고, 메인쿼리가 그 여러 컬럼을 **묶어서 동시에** 비교하는 형태다. `(컬럼1, 컬럼2) IN (SELECT 컬럼1, 컬럼2 ...)` 모양이다.

**2. 왜 필요한가?**
"각 부서에서 가장 월급 높은 사람", "각 앨범에서 가장 긴 트랙"처럼 **그룹별 대표 행**을 뽑을 때다. (부서, 최대급여) 쌍을 한꺼번에 맞춰야 정확히 그 사람만 나온다.

**3. 예시 (각 앨범에서 가장 긴 트랙)**
```sql
SELECT Name, AlbumId, Milliseconds
FROM tracks
WHERE (AlbumId, Milliseconds) IN (
    SELECT AlbumId, MAX(Milliseconds)
    FROM tracks
    GROUP BY AlbumId
)
ORDER BY AlbumId
LIMIT 15;
```

**4. 헷갈리기 쉬운 점**
`Milliseconds = (SELECT MAX(Milliseconds) FROM tracks)`처럼 한 컬럼만 비교하면, "전체에서 가장 긴 트랙"이 되어 버린다. **앨범별** 최대를 원한다면 `(AlbumId, MAX(...))` 쌍을 묶어 비교해야 "각 앨범의 최댓값과 정확히 일치하는 행"만 나온다.

**5. 한 줄 정리**
다중 컬럼 서브쿼리는 여러 컬럼을 묶어 비교해 그룹별 대표 행을 정확히 집어낸다.

> 비유: "이름 + 생년월일"을 함께 대조해야 동명이인 중 진짜 그 사람을 찾는 것.

## 코드로 보기 — SQLite에서 ALL / ANY 대체하기

SQLite는 `ALL`, `ANY` 연산자를 지원하지 않는다. 대신 동치 관계를 이용해 `MAX`/`MIN` 서브쿼리로 바꾼다.

```sql
-- ALL 대체: "Rock 최장 트랙보다도 긴 트랙" → >= ALL ≡ > (MAX)
SELECT Name, Milliseconds
FROM tracks
WHERE Milliseconds > (
    SELECT MAX(t.Milliseconds)            -- ALL 대체: MAX
    FROM tracks t
    INNER JOIN genres g ON t.GenreId = g.GenreId
    WHERE g.Name = 'Rock'
)
ORDER BY Milliseconds DESC
LIMIT 10;

-- ANY 대체: "Classical 최단 트랙보다 긴 트랙" → > ANY ≡ > (MIN)
SELECT Name, Milliseconds
FROM tracks
WHERE Milliseconds > (
    SELECT MIN(t.Milliseconds)            -- ANY 대체: MIN
    FROM tracks t
    INNER JOIN genres g ON t.GenreId = g.GenreId
    WHERE g.Name = 'Classical'
)
ORDER BY Milliseconds
LIMIT 10;
```

**코드목적**
표준 SQL의 `ALL`/`ANY`를, 지원하지 않는 환경에서 `MAX`/`MIN` 서브쿼리로 동일하게 구현한다.

**해석**
"모든 값보다 크다(ALL)"는 곧 "그 집합의 최댓값보다 크다"와 같고, "어떤 값보다 크다(ANY)"는 "그 집합의 최솟값보다 크다"와 같다. 이 논리적 동치를 알면, `ALL`/`ANY`가 없는 SQLite에서도 같은 의미의 쿼리를 만들 수 있다.

**실무 연결**
DB 종류마다 지원 문법이 조금씩 다르다. "안 되면 같은 의미의 다른 문법으로 바꾸는" 습관은 여러 DB를 오갈 때 꼭 필요하다.

## 직접 해보기

1. (단일 행) 가장 짧은 트랙의 재생시간 이하인 트랙을 모두 조회해 보세요. (`MIN` 서브쿼리)
2. (다중 행) 단 한 번도 판매되지 않은 트랙의 이름을 `NOT IN`으로 조회해 보세요. (`invoice_items`에 없는 `TrackId`)
3. (다중 컬럼) 각 장르(`GenreId`)에서 단가(`UnitPrice`)가 가장 낮은 트랙을 `(GenreId, MIN(UnitPrice))` 쌍으로 조회해 보세요.

## 헷갈리기 쉬운 포인트

- **단일 행 vs 다중 행**: `=`·`>`엔 1행만, `IN`·`EXISTS`엔 여러 행. 단일 행 연산자에 여러 행이 오면 에러.
- **IN vs EXISTS**: IN은 값 목록 비교, EXISTS는 존재 확인. NULL 섞인 `NOT IN`은 위험 → `NOT EXISTS` 권장.
- **`>= ALL` vs `>= ANY`**: ALL은 최댓값 기준, ANY는 최솟값 기준. SQLite에선 각각 `MAX`/`MIN`으로 대체.
- **한 컬럼 vs 다중 컬럼 비교**: 그룹별 대표 행은 반드시 `(그룹키, 집계값)` 쌍으로 묶어 비교.

## 연결되는 개념

- 이전 글: [⑤ 서브쿼리 ① 동작 방식](05-subquery-correlated.md) — 연관/비연관
- 다음 글: [⑦ 스칼라 서브쿼리](07-scalar-subquery.md) — 값 하나를 SELECT 자리에 끼워넣기
- 함께 보면 좋은: [③ OUTER JOIN](03-outer-join.md)의 `IS NULL` — "없는 것 찾기"의 또 다른 길
- 더 찾아볼 키워드: `NOT EXISTS`, `상위 N개(그룹별 Top-N)`, `윈도우 함수`

## 셀프 체크

- [ ] 서브쿼리를 결과 모양(단일/다중행/다중컬럼)으로 분류할 수 있다.
- [ ] 단일 행 연산자와 다중 행 연산자를 구분해 쓸 수 있다.
- [ ] `IN`/`EXISTS`/`ALL`/`ANY`의 의미를 설명할 수 있다.
- [ ] 다중 컬럼 서브쿼리로 그룹별 대표 행을 뽑을 수 있다.
- [ ] SQLite에서 `ALL`/`ANY`를 `MAX`/`MIN`으로 대체할 수 있다.

**복습 질문 및 답변**

- (기본) 서브쿼리가 여러 행을 반환하는데 `=`로 비교하면 어떻게 되나요?
  → 단일 행 연산자에 여러 행이 들어와 에러가 납니다. `IN` 등 다중 행 연산자를 써야 합니다.
- (이해 확인) `>= ALL`은 어떤 단일 값 비교와 같은 의미인가요?
  → 그 집합의 최댓값과 비교하는 것(`>= (SELECT MAX(...))`)과 같습니다.
- (응용) "각 앨범에서 가장 긴 트랙"을 뽑을 때 `Milliseconds = (SELECT MAX(Milliseconds) FROM tracks)`가 틀린 이유는?
  → 그건 전체에서 가장 긴 트랙을 뜻합니다. 앨범별 최대를 원하면 `(AlbumId, MAX(Milliseconds))` 쌍으로 비교해야 합니다.

## 한 줄 정리

> 서브쿼리는 결과가 값 하나면 단일 행 연산자, 여러 행이면 다중 행 연산자, 여러 컬럼이면 묶음 비교를 써야 하며, 형태와 연산자를 맞추는 것이 핵심이다.
