# 스칼라 서브쿼리 — 값 하나를 결과에 끼워넣기

> "부서명 옆에 그 부서 인원수도 같이 보여 줘." 인원수는 다른 표를 세어야 나오는 값이죠. 이렇게 **한 칸에 들어갈 값 하나**를 즉석에서 계산해 넣는 것이 스칼라 서브쿼리입니다.

`스칼라서브쿼리` `Scalar Subquery` `SELECT절서브쿼리` `단일값` `HAVING` `비율계산`

## 핵심요약

- **스칼라 서브쿼리**는 **하나의 컬럼 + 하나의 행**, 즉 **값 하나(스칼라)**만 반환하는 서브쿼리다.
- `SELECT`, `WHERE`, `HAVING` 절 등에서 **값이 들어갈 자리**에 끼워 쓸 수 있다.
- `SELECT`절에 쓰면 결과 표에 **새 컬럼**처럼 계산값이 붙는다.
- 흔히 메인쿼리의 현재 행을 참조하는 **연관 서브쿼리** 형태로 쓰인다.
- 반드시 값 하나만 나와야 한다 — 여러 행이 나오면 에러.
- SQLite엔 `DUAL` 테이블이 없으므로, 테이블 없는 계산은 **`FROM` 없이 `SELECT`만** 작성한다.

## 개념별 정리

### 스칼라 서브쿼리란

**1. 정의**
결과가 단 하나의 값(스칼라)인 서브쿼리다. "스칼라"는 행렬·목록이 아닌 **단일 값**을 뜻한다.

**2. 왜 필요한가?**
조회 결과의 각 행 옆에 "관련된 계산값 하나"를 붙이고 싶을 때가 많다. 부서명 옆 인원수, 아티스트명 옆 앨범 수처럼. 값이 들어갈 자리에 서브쿼리를 그대로 넣을 수 있어 표현이 깔끔하다.

**3. 예시 (장르별 트랙 수)**
```sql
SELECT g.Name AS GenreName,
       (
           SELECT COUNT(*)
           FROM tracks t
           WHERE t.GenreId = g.GenreId    -- 메인쿼리 g.GenreId 참조
       ) AS TrackCount
FROM genres g
ORDER BY TrackCount DESC;
```
`SELECT`절 안의 괄호 서브쿼리가 각 장르마다 트랙 수를 세어, `TrackCount`라는 컬럼처럼 결과에 붙는다.

**4. 헷갈리기 쉬운 점**
스칼라 서브쿼리는 반드시 **값 하나**만 내야 한다. 만약 서브쿼리가 여러 행을 반환하면 "값 하나가 들어갈 자리에 여러 개가 왔다"며 에러가 난다. `COUNT`, `SUM`, `MAX` 같은 집계나 1행만 보장되는 조건을 써야 한다.

**5. 한 줄 정리**
스칼라 서브쿼리는 값 하나를 만들어, 값이 들어갈 자리에 끼워 넣는 서브쿼리다.

> 비유: 표의 빈칸 하나하나에 "그 행에 맞는 계산값"을 채워 넣는 자동 채움.

### SELECT절 vs WHERE절에서의 활용

**1. 정의**
스칼라 서브쿼리는 값이 필요한 여러 자리에 들어간다. `SELECT`절에 쓰면 **새 컬럼**으로 보이고, `WHERE`절에 쓰면 **필터 기준값**이 된다.

**2. 왜 필요한가?**
"보여 줄 값"으로 쓸지, "거를 기준"으로 쓸지에 따라 위치만 바꾸면 된다. 같은 서브쿼리를 목적에 맞게 재활용할 수 있다.

**3. 예시 (평균보다 많이 지출한 고객 — WHERE절)**
```sql
SELECT c.FirstName || ' ' || c.LastName AS CustomerName,
       (SELECT ROUND(SUM(i.Total), 2)
        FROM invoices i
        WHERE i.CustomerId = c.CustomerId)   AS TotalSpent          -- SELECT절 스칼라
FROM customers c
WHERE (SELECT COALESCE(SUM(i.Total), 0)
       FROM invoices i
       WHERE i.CustomerId = c.CustomerId)                            -- WHERE절 스칼라
      >
      (SELECT AVG(customer_total)
       FROM (SELECT SUM(Total) AS customer_total
             FROM invoices GROUP BY CustomerId))
ORDER BY TotalSpent DESC;
```

**4. 헷갈리기 쉬운 점**
지출 내역이 없는 고객은 `SUM`이 NULL이 된다. `WHERE`에서 비교가 어긋나지 않도록 `COALESCE(..., 0)`로 NULL을 0으로 바꿔 주는 안전장치가 들어 있다. NULL을 그대로 두면 비교가 참도 거짓도 아니게 되어 행이 빠질 수 있다.

**5. 한 줄 정리**
스칼라 서브쿼리는 SELECT절에선 새 컬럼, WHERE절에선 기준값으로 같은 문법을 재활용한다.

### DUAL 없이 계산하기

**1. 정의**
오라클 등에는 "행이 하나뿐인 더미 테이블" `DUAL`이 있어 테이블 없이 계산식만 쓸 때 `FROM DUAL`을 붙인다. SQLite엔 DUAL이 없으니 **`FROM`을 아예 생략**한다.

**2. 왜 필요한가?**
"전체 중 특정 그룹의 비율"처럼 특정 테이블을 훑을 필요 없이 **계산 결과 한 줄**만 내고 싶을 때 쓴다.

**3. 예시 (전체 트랙 중 Rock 비율)**
```sql
SELECT ROUND(
    CAST((SELECT COUNT(*) FROM tracks t
          INNER JOIN genres g ON t.GenreId = g.GenreId
          WHERE g.Name = 'Rock') AS REAL)
    / CAST((SELECT COUNT(*) FROM tracks) AS REAL),
    4
) AS RockRatio;
```

**4. 헷갈리기 쉬운 점**
정수끼리 나누면 소수점이 잘려 0이 되는 환경이 있다. 그래서 `CAST(... AS REAL)`로 실수 변환을 해 준 뒤 나눈다. 비율·평균 계산에서 자주 놓치는 지점이다.

**5. 한 줄 정리**
SQLite에선 `FROM` 없이 `SELECT`만 써서 값 하나를 계산할 수 있다.

## 코드로 보기 — 부서(장르)명과 구성원 수

```sql
SELECT g.Name AS GenreName,
       (
           SELECT COUNT(*)
           FROM tracks t
           WHERE t.GenreId = g.GenreId
       ) AS TrackCount
FROM genres g
ORDER BY TrackCount DESC;
```

**코드목적**
강의 슬라이드의 "부서명과 부서 구성원 수" 패턴을 그대로 옮겨, 각 장르 이름 옆에 해당 장르 트랙 수를 함께 출력한다.

**해석**
메인쿼리는 장르를 한 줄씩 본다. 각 장르마다 `SELECT`절의 스칼라 서브쿼리가 "이 장르에 속한 트랙 수"를 세어 새 컬럼처럼 붙인다. 서브쿼리 안의 `g.GenreId`가 현재 보고 있는 장르를 가리키므로 행마다 다른 값이 채워진다(연관 서브쿼리이기도 하다).

**실무 연결**
목록에 "관련 집계 한 칸"을 덧붙이는 건 대시보드·관리자 화면의 단골 패턴이다(카테고리별 상품 수, 사용자별 게시글 수 등). JOIN + GROUP BY로도 가능하지만, "한 컬럼만 덧붙일 때"는 스칼라 서브쿼리가 더 간결할 때가 많다.

## 직접 해보기

1. 각 아티스트 이름과 그 아티스트의 총 앨범 수를 스칼라 서브쿼리로 함께 조회해 보세요.
2. 전체 트랙 중 `'Rock'` 장르가 차지하는 비율을 소수점 4자리로 출력해 보세요. (`FROM` 없이, `CAST`로 실수 변환)
3. 전체 고객 평균 지출보다 많이 쓴 고객의 이름과 총 지출액을 조회해 보세요. (지출 없는 고객은 `COALESCE`로 0 처리)

## 헷갈리기 쉬운 포인트

- **스칼라 서브쿼리 vs 다중 행 서브쿼리**: 스칼라는 값 하나만, 다중 행은 여러 행. 스칼라가 여러 행을 내면 에러.
- **SELECT절 서브쿼리 vs JOIN+GROUP BY**: 결과가 비슷할 수 있으나, 한 컬럼만 덧붙일 땐 스칼라가 간결, 여러 집계가 필요하면 JOIN+GROUP BY가 유리.
- **정수 나눗셈 함정**: 비율 계산 시 `CAST(... AS REAL)`을 빼면 0이 나올 수 있다.
- **NULL 처리**: 집계가 NULL이 될 수 있는 자리엔 `COALESCE(..., 0)`로 방어.

## 연결되는 개념

- 이전 글: [⑥ 서브쿼리 ② 반환 형태](06-subquery-return-shape.md) — 단일 행 서브쿼리의 연장선
- 다음 글: [⑧ 뷰(VIEW)](08-view.md) — 복잡한 쿼리를 표처럼 재사용
- 함께 보면 좋은: [⑤ 서브쿼리 ① 동작 방식](05-subquery-correlated.md) — 스칼라 서브쿼리는 주로 연관 형태
- 더 찾아볼 키워드: `COALESCE`, `CAST`, `상관 스칼라 서브쿼리`, `윈도우 함수`

## 셀프 체크

- [ ] 스칼라 서브쿼리가 "값 하나"만 반환함을 안다.
- [ ] `SELECT`절 스칼라 서브쿼리가 새 컬럼처럼 붙는 것을 안다.
- [ ] 스칼라 서브쿼리가 여러 행을 내면 에러임을 안다.
- [ ] SQLite에서 `DUAL` 대신 `FROM`을 생략함을 안다.
- [ ] 비율 계산에서 `CAST(... AS REAL)`이 필요한 이유를 안다.

**복습 질문 및 답변**

- (기본) 스칼라 서브쿼리는 결과가 어떤 모양이어야 하나요?
  → 한 컬럼·한 행, 즉 값 하나여야 합니다.
- (이해 확인) `SELECT`절에 스칼라 서브쿼리를 넣으면 결과에 어떻게 나타나나요?
  → 계산된 값이 하나의 새 컬럼처럼 각 행 옆에 붙습니다.
- (응용) 정수 두 개를 나눠 비율을 구했더니 0이 나왔습니다. 원인과 해결은?
  → 정수 나눗셈으로 소수점이 잘렸기 때문입니다. `CAST(... AS REAL)`로 실수 변환 후 나누면 됩니다.

## 한 줄 정리

> 스칼라 서브쿼리는 값 하나를 계산해 SELECT·WHERE·HAVING의 "값 자리"에 끼워 넣는 서브쿼리다.
