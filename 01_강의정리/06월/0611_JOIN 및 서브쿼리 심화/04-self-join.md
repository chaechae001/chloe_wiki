# 셀프 조인 — 한 표를 둘로 나눠 비교하기

> "직원 표 하나에 사원번호와 그 사람의 관리자 번호가 같이 들어 있어요. 그럼 '누가 누구의 상사인지'를 어떻게 한 줄로 보죠?" — 같은 표를 두 번 불러오는 셀프 조인이 답입니다.

`셀프조인` `Self Join` `별칭` `계층형질의` `자기참조` `쌍비교`

## 핵심요약

- **셀프 조인**은 한 테이블을 **자기 자신과 조인**하는 것이다.
- 같은 표를 두 번 쓰면 테이블·컬럼 이름이 겹치므로, **별칭(alias)이 필수**다.
- 대표 용도 ①: **계층 구조**(상사–부하, 조직도, 카테고리 부모–자식) 풀어내기.
- 대표 용도 ②: **같은 집합 안의 쌍 비교**(같은 국가 고객끼리, 같은 앨범 트랙끼리).
- 쌍을 만들 때는 `A.키 < B.키` 조건으로 **(A,B)와 (B,A) 중복 쌍을 제거**한다.
- 자기참조 외래키가 없어도, **공통 그룹키(앨범ID·국가 등)**로 묶어 셀프 조인할 수 있다.

## 개념별 정리

### 셀프 조인이란

**1. 정의**
물리적으로는 한 개의 테이블이지만, 별칭을 두 개 붙여 **마치 서로 다른 두 표인 것처럼** 조인하는 기법이다.

**2. 왜 필요한가?**
한 표 안에 "행끼리의 관계"가 들어 있는 경우가 있다. 직원 표의 각 행이 자기 관리자(다른 행)의 번호를 들고 있는 식이다. 이런 행–행 관계를 펼치려면 같은 표를 두 번 불러 한쪽은 "직원", 다른 쪽은 "상사"로 보면 된다.

**3. 예시 (직원과 직속 관리자)**
```sql
SELECT e.FirstName || ' ' || e.LastName  AS EmployeeName,
       e.Title                           AS EmployeeTitle,
       m.FirstName || ' ' || m.LastName  AS ManagerName,
       m.Title                           AS ManagerTitle
FROM employees e
LEFT JOIN employees m ON e.ReportsTo = m.EmployeeId
ORDER BY e.EmployeeId;
```
`e`는 직원, `m`은 그 직원의 관리자로 본다. `e.ReportsTo`(관리자 번호)와 `m.EmployeeId`(상사의 사원번호)를 맞춘다.

**4. 헷갈리기 쉬운 점**
별칭을 빼면 "어느 쪽 EmployeeId인지" 구분이 안 돼 오류가 나거나 의도와 다른 결과가 된다. 셀프 조인에서 별칭은 선택이 아니라 필수다. 또 최상위 관리자는 상사가 없으므로(`ReportsTo`가 NULL) **LEFT JOIN**을 써야 그 사람도 결과에 남는다.

**5. 한 줄 정리**
셀프 조인은 한 표에 별칭 두 개를 붙여 자기 자신과 잇는, 별칭 필수의 조인이다.

> 비유: 한 장의 가족 사진을 두 번 복사해 "부모 역할"과 "자녀 역할"로 나눠 놓고 짝을 맞추는 것.

### 계층 한 단계 더 — 차상위 관리자

**1. 정의**
직원 → 직속 상사 → 그 상사의 상사(2단계)까지 펼치려면, 셀프 조인을 **한 번 더** 연결한다.

**2. 왜 필요한가?**
조직도는 보통 여러 단계다. "내 위의 위 사람은 누구인가"를 보려면 계층을 한 단계씩 확장해야 한다.

**3. 예시**
```sql
SELECT e.FirstName || ' ' || e.LastName  AS Employee,
       m.FirstName || ' ' || m.LastName  AS DirectManager,
       g.FirstName || ' ' || g.LastName  AS GrandManager
FROM employees e
LEFT JOIN employees m ON e.ReportsTo  = m.EmployeeId
LEFT JOIN employees g ON m.ReportsTo  = g.EmployeeId
ORDER BY e.EmployeeId;
```
`e`(직원) → `m`(직속 상사) → `g`(차상위 상사)로 별칭을 세 개 두고 한 단계씩 타고 올라간다.

**4. 헷갈리기 쉬운 점**
단계마다 LEFT JOIN을 쓰는 이유는, 위가 없는 사람(예: 최상위 직원의 차상위)이 중간에 끊겨도 결과에서 사라지지 않게 하기 위함이다. INNER JOIN으로 하면 상위가 없는 직원이 통째로 빠진다.

**5. 한 줄 정리**
계층을 더 타려면 셀프 조인을 한 번씩 더 이어 붙이면 된다.

### 같은 그룹 안의 쌍 비교

**1. 정의**
같은 그룹(같은 국가, 같은 앨범 등)에 속한 행끼리 짝지어 비교하는 셀프 조인이다.

**2. 왜 필요한가?**
"같은 국가에 사는 고객 쌍", "같은 앨범에서 재생시간이 비슷한 트랙 쌍"처럼 **동일 집합 내 비교**가 필요할 때 쓴다. 추천(비슷한 항목 찾기), 중복 탐지 등에 쓰인다.

**3. 예시 (같은 앨범 내 재생시간이 비슷한 트랙 쌍)**
```sql
SELECT t1.Name        AS TrackA,
       t2.Name        AS TrackB,
       t1.Milliseconds AS MsA,
       t2.Milliseconds AS MsB,
       ABS(t1.Milliseconds - t2.Milliseconds) AS MsDiff
FROM tracks t1
JOIN tracks t2
  ON t1.AlbumId   =  t2.AlbumId          -- 같은 앨범끼리
 AND t1.TrackId   <  t2.TrackId          -- 중복 쌍 제거
 AND ABS(t1.Milliseconds - t2.Milliseconds) <= 60000
ORDER BY MsDiff
LIMIT 15;
```

**4. 헷갈리기 쉬운 점**
조건 없이 같은 그룹끼리 묶으면 (A,A) 자기 자신 쌍과 (A,B)·(B,A) 중복 쌍이 모두 생긴다. `t1.TrackId < t2.TrackId`처럼 **"작은 쪽을 A"** 조건을 넣으면 자기 쌍과 뒤집힌 중복 쌍이 한 번에 제거된다. 여기서 조인 키는 자기참조 외래키가 아니라 **공통 그룹키(AlbumId)**라는 점도 포인트다.

**5. 한 줄 정리**
같은 그룹의 쌍 비교는 셀프 조인 + `작은키 < 큰키`로 중복 없이 만든다.

> 비유: 같은 동네 사람들끼리 악수시키되, (철수–영희)와 (영희–철수)를 한 번만 세는 규칙.

## 코드로 보기 — 같은 국가 고객 쌍

```sql
SELECT c1.FirstName || ' ' || c1.LastName AS CustomerA,
       c2.FirstName || ' ' || c2.LastName AS CustomerB,
       c1.Country
FROM customers c1
JOIN customers c2
  ON c1.Country    = c2.Country          -- 같은 국가
 AND c1.CustomerId < c2.CustomerId        -- 중복 쌍 제거 (작은 쪽이 A)
ORDER BY c1.Country, CustomerA
LIMIT 20;
```

**코드목적**
같은 국가에 속한 고객들을 둘씩 짝지어, 같은 쌍이 두 번 나오지 않게 목록으로 만든다.

**해석**
`customers`를 `c1`, `c2` 두 별칭으로 불러 같은 국가 조건(`c1.Country = c2.Country`)으로 잇는다. 이대로면 (A,B)와 (B,A)가 둘 다 나오므로, `c1.CustomerId < c2.CustomerId` 조건으로 "번호가 작은 쪽을 항상 A"로 고정해 중복을 없앤다. 자기 자신과의 쌍(A,A)도 이 부등호 덕분에 자동 제외된다.

**실무 연결**
"같은 지역 고객 매칭", "동일 카테고리 상품 비교", "중복 의심 레코드 찾기" 등 동일 집합 내 관계 분석은 전부 이 셀프 조인 패턴이다.

## 직접 해보기

1. `employees`에서 각 직원의 이름과 직속 관리자 이름을 조회하되, 관리자가 없는 최상위 직원도 포함되게(LEFT JOIN) 해 보세요.
2. 직원 → 직속 상사 → 차상위 상사를 한 줄에 보여 주는 3단계 셀프 조인을 작성해 보세요.
3. 같은 국가 고객 쌍을 만들되, `CustomerId`가 작은 쪽을 A로 두어 중복 쌍이 없도록 상위 20행만 조회해 보세요.

## 헷갈리기 쉬운 포인트

- **별칭 있음 vs 없음**: 셀프 조인은 별칭이 없으면 컬럼 구분이 안 돼 동작하지 않는다. 별칭은 필수.
- **INNER vs LEFT (계층형에서)**: 최상위처럼 상위가 없는 행을 남기려면 LEFT JOIN. INNER로 하면 그 행이 사라진다.
- **중복 쌍 제거: `<` vs `!=`**: `!=`만 쓰면 (A,B)·(B,A) 둘 다 남는다. `<`(또는 `>`)를 써야 한 방향만 남아 중복이 제거된다.
- **자기참조 FK vs 공통 그룹키**: 계층형은 자기참조 키(`ReportsTo`)로, 쌍 비교는 공통 그룹키(`Country`·`AlbumId`)로 잇는다.

## 연결되는 개념

- 이전 글: [③ OUTER JOIN](03-outer-join.md) — 계층형 셀프 조인에서 LEFT JOIN이 쓰이는 이유
- 다음 글: [⑤ 서브쿼리 ① 동작 방식](05-subquery-correlated.md) — 같은 표를 참조하는 또 다른 방법
- 더 찾아볼 키워드: `계층형 질의`, `재귀 CTE(WITH RECURSIVE)`, `자기참조 외래키`

## 셀프 체크

- [ ] 셀프 조인이 "한 표를 별칭 두 개로 잇는 것"임을 안다.
- [ ] 셀프 조인에서 별칭이 필수인 이유를 설명할 수 있다.
- [ ] 계층형(상사–부하)을 셀프 조인으로 풀 수 있다.
- [ ] 최상위 행을 남기려면 LEFT JOIN을 써야 함을 안다.
- [ ] 쌍 비교에서 `작은키 < 큰키`로 중복을 제거할 수 있다.

**복습 질문 및 답변**

- (기본) 셀프 조인에서 별칭이 꼭 필요한 이유는?
  → 같은 표를 두 번 쓰면 테이블명·컬럼명이 겹쳐 구분이 불가능하기 때문입니다.
- (이해 확인) 직원–관리자 조회에서 INNER JOIN 대신 LEFT JOIN을 쓰는 이유는?
  → 관리자가 없는 최상위 직원(`ReportsTo`가 NULL)을 결과에 남기기 위해서입니다.
- (응용) 같은 국가 고객 쌍에서 (A,B)와 (B,A) 중복을 한 번에 없애려면?
  → `c1.CustomerId < c2.CustomerId` 같은 부등호 조건으로 한쪽 방향만 남깁니다.

## 한 줄 정리

> 셀프 조인은 한 표를 별칭 두 개로 잇는 기법으로, 계층 구조 풀기와 같은 그룹 내 쌍 비교에 쓰인다.
