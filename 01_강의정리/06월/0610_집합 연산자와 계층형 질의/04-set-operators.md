# 집합 연산자 — 여러 결과를 위아래로 합치고 빼기

> 조인이 "두 표를 옆으로(가로) 잇는 것"이라면, 집합 연산자는 "두 표를 위아래로(세로) 쌓거나 빼는 것"입니다. 명단 합치기·교집합·이탈 고객 찾기가 모두 여기에 해당합니다.

`집합 연산자` `UNION` `UNION ALL` `INTERSECT` `EXCEPT` `MINUS` `중복 제거` `소계 행` `DBMS 차이`

## 핵심요약

- **집합 연산자**는 조인을 쓰지 않고, 두 개 이상의 `SELECT` 결과를 *위아래로* 묶는 방법이다.
- 네 가지: `UNION`(합집합·중복 제거), `UNION ALL`(합집합·중복 유지), `INTERSECT`(교집합), `EXCEPT`(차집합).
- 전제 조건: **컬럼 수가 같아야** 하고, **대응 컬럼의 데이터 타입이 호환**되어야 한다.
- 결과 컬럼명은 **첫 번째 쿼리**의 컬럼명을 따르고, `ORDER BY`는 **최종 결합 결과**에만 한 번 적용한다.
- `UNION`은 중복 제거를 위해 정렬을 동반하지만, `UNION ALL`은 정렬·중복 제거를 하지 않아 더 빠르다.
- DBMS마다 지원이 다르다. 차집합은 Oracle에서 `MINUS`, MariaDB·SQLite·SQL Server에서 `EXCEPT`. MySQL은 `INTERSECT`/`EXCEPT` 미지원이라 `JOIN`으로 대체한다.

## 개념별 정리

### 집합 연산자란?

**1. 정의**
두 개 이상의 테이블(또는 쿼리 결과)에서, 조인 대신 *결과를 세로로 결합*해 연관 데이터를 조회하는 방법입니다.

**2. 왜 필요한가?**
"직원 명단과 고객 명단을 하나의 이름 목록으로", "두 해 모두 구매한 고객만"처럼, 옆으로 잇는 조인으로는 어색한 요구를 자연스럽게 표현할 수 있습니다.

**3. 예시**
강의 슬라이드의 ALPHA/BETA 표로 동작을 정리하면 다음과 같습니다.

- ALPHA: `(A1,B1,1) (A2,B2,2) (A3,B3,3)`
- BETA: `(A1,B1,1) (A4,B4,4)`

| 연산자 | 결과 | 중복 제거 |
| --- | --- | --- |
| `UNION` | A1·A2·A3·A4 | O |
| `UNION ALL` | A1·A2·A3·A1·A4 (A1 두 번) | X |
| `INTERSECT` | A1 | O |
| `EXCEPT` | A2·A3 (ALPHA에만 있는 행) | O |

**4. 헷갈리기 쉬운 점**
집합 연산은 컬럼 *이름*이 같은지를 보지 않고, 컬럼 *개수와 순서, 데이터 타입*을 봅니다. 첫 쿼리는 컬럼 3개인데 둘째 쿼리가 2개면 "결과 컬럼 수가 다르다"는 오류가 납니다.

**5. 한 줄 정리**
집합 연산자는 "조인 없이 결과를 세로로 묶는" 도구다.

> 비유: 두 권의 주소록을 한 권으로 합치거나(합집합), 양쪽 모두에 있는 사람만 추리거나(교집합), 한쪽에만 있는 사람을 찾는 것(차집합)입니다.

### UNION — 합집합 (중복 제거)

**1. 정의**
두 결과를 하나로 합친 뒤, *중복된 행을 제거*합니다.

**2. 왜 필요한가?**
서로 다른 테이블에 흩어진 같은 종류의 데이터(예: 직원 이름 + 고객 이름)를 하나의 목록으로 만들 때 씁니다.

**3. 예시**

```sql
-- 직원 이름 + 고객 이름을 하나의 목록으로 (출처 표시 포함)
SELECT FirstName, LastName, 'Employee' AS Type
FROM employees
UNION
SELECT FirstName, LastName, 'Customer'
FROM customers
ORDER BY FirstName, LastName;
```

`'Employee'`, `'Customer'`처럼 상수 컬럼을 추가하면 합친 뒤에도 *어느 표에서 왔는지* 구분할 수 있습니다.

**4. 헷갈리기 쉬운 점**
`UNION`은 중복 제거 과정에서 정렬이 일어나지만, *최종 결과의 정렬 순서를 보장하지는 않습니다.* 원하는 순서가 있다면 맨 끝에 `ORDER BY`를 명시해야 합니다.

**5. 한 줄 정리**
UNION은 "합치고 중복은 지운다."

> 비유: 두 동아리의 회원 명단을 합치되, 양쪽에 모두 가입한 사람은 한 번만 적는 것입니다.

### UNION ALL — 합집합 (중복 유지, 더 빠름)

**1. 정의**
`UNION`과 거의 같지만, *중복 제거와 정렬을 하지 않습니다.* 그래서 더 빠릅니다.

**2. 왜 필요한가?**
중복이 없다는 게 확실하거나 중복을 일부러 남기고 싶을 때, 또 "집계 결과 + 전체 합계 행"처럼 *소계/합계 행을 덧붙일 때* 자주 씁니다.

**3. 예시**

```sql
-- 연도별 매출 + 맨 아래에 전체 합계 행 붙이기
SELECT strftime('%Y', InvoiceDate) AS 연도,
       ROUND(SUM(Total), 2)        AS 매출합계
FROM invoices
GROUP BY 연도
UNION ALL
SELECT '── 전체합계',
       ROUND(SUM(Total), 2)
FROM invoices
ORDER BY 연도;
```

**4. 헷갈리기 쉬운 점**
중복 제거를 하지 않는다는 점이 `UNION`과의 *가장 중요한 차이*입니다. 데이터가 클 때 무심코 `UNION`을 쓰면 불필요한 정렬·중복 제거 비용이 들 수 있으니, 중복이 없다면 `UNION ALL`이 낫습니다.

**5. 한 줄 정리**
UNION ALL은 "합치되 그대로 둔다(중복·정렬 손대지 않음)."

> 비유: 두 명단을 그냥 이어 붙이는 것. 같은 사람이 양쪽에 있으면 두 번 적힙니다.

### INTERSECT — 교집합

**1. 정의**
두 결과에서 *양쪽에 모두 있는 행*만 남기고, 중복은 제거합니다.

**2. 왜 필요한가?**
"두 해 모두 구매한 고객", "A 조건과 B 조건을 동시에 만족하는 대상"을 뽑을 때 직관적입니다.

**3. 예시**

```sql
-- 2009년과 2010년에 모두 구매한 고객 ID
SELECT CustomerId
FROM invoices
WHERE strftime('%Y', InvoiceDate) = '2009'
INTERSECT
SELECT CustomerId
FROM invoices
WHERE strftime('%Y', InvoiceDate) = '2010'
ORDER BY CustomerId;
```

이 결과(고객 ID)를 고객 테이블과 조인하면 이름까지 함께 볼 수 있습니다.

**4. 헷갈리기 쉬운 점**
`INTERSECT`는 MySQL에서 지원되지 않습니다. 같은 결과는 `INNER JOIN`이나 `IN` 서브쿼리로 대체할 수 있습니다.

**5. 한 줄 정리**
INTERSECT는 "양쪽에 다 있는 것만" 남긴다.

> 비유: 두 명단을 겹쳐 놓고, 양쪽에 모두 적힌 이름만 골라내는 것입니다.

### EXCEPT (MINUS) — 차집합

**1. 정의**
앞 결과에서 *뒤 결과에 있는 행을 제외*하고, 중복은 제거합니다.

**2. 왜 필요한가?**
"작년엔 샀지만 올해는 안 산 고객"(이탈 고객), "전체 고객 중 청구서가 없는 고객"처럼 *한쪽에만 있는 대상*을 뽑을 때 씁니다.

**3. 예시**

```sql
-- 2009년엔 구매했지만 2010년엔 구매하지 않은 고객 (이탈 고객)
SELECT CustomerId
FROM invoices
WHERE strftime('%Y', InvoiceDate) = '2009'
EXCEPT
SELECT CustomerId
FROM invoices
WHERE strftime('%Y', InvoiceDate) = '2010'
ORDER BY CustomerId;
```

**4. 헷갈리기 쉬운 점**
순서가 중요합니다. `A EXCEPT B`는 "A에는 있고 B에는 없는 것"이라, 앞뒤를 바꾸면 결과가 완전히 달라집니다. 또 데이터에 따라 결과가 비어 있을 수도 있는데(예: 모든 고객에게 청구서가 있는 경우), 이는 오류가 아니라 *그런 대상이 없다는 뜻*입니다.

**5. 한 줄 정리**
EXCEPT는 "앞에는 있고 뒤에는 없는 것"을 남긴다.

> 비유: 작년 회원 명단에서 올해 명단에 있는 이름을 지우면, "올해 떠난 회원"만 남는 것과 같습니다.

### DBMS별 지원 차이

**1. 정의**
같은 집합 연산이라도, 데이터베이스 제품마다 키워드나 지원 여부가 다릅니다.

**2. 왜 필요한가?**
배운 문법이 다른 환경에서 안 먹힐 수 있어, *어느 DBMS에서 무엇이 되는지*를 알아 두면 실무 이전이 매끄럽습니다.

**3. 예시**

| 연산 | Oracle | SQL Server | MariaDB | SQLite | MySQL |
| --- | --- | --- | --- | --- | --- |
| 합집합 | `UNION` / `UNION ALL` | 동일 | 동일 | 동일 | 동일 |
| 교집합 | `INTERSECT` | `INTERSECT` | `INTERSECT` | `INTERSECT` | 미지원 → `JOIN` |
| 차집합 | `MINUS` | `EXCEPT` | `EXCEPT`(10.3+) | `EXCEPT` | 미지원 → `JOIN` |

**4. 헷갈리기 쉬운 점**
차집합 키워드가 Oracle은 `MINUS`, 다른 곳은 대체로 `EXCEPT`로 갈립니다. MySQL은 교집합·차집합 자체가 없어 조인으로 우회해야 합니다.

**5. 한 줄 정리**
개념은 표준이지만, 키워드는 DBMS마다 다를 수 있다.

> 비유: "빼기"라는 개념은 같아도, 나라마다 부르는 말(MINUS / EXCEPT)이 다른 것과 같습니다.

## 코드로 보기 — 구매 이력 유무로 고객 분류하기

```sql
-- 구매한 고객과 구매 안 한 고객을 한 목록으로 (상태 표시)
SELECT CustomerId,
       FirstName || ' ' || LastName AS 고객명,
       'Has Invoice'                AS 상태
FROM customers
WHERE CustomerId IN (SELECT DISTINCT CustomerId FROM invoices)

UNION

SELECT CustomerId,
       FirstName || ' ' || LastName AS 고객명,
       'No Invoice'                 AS 상태
FROM customers
WHERE CustomerId NOT IN (SELECT DISTINCT CustomerId FROM invoices)

ORDER BY CustomerId;
```

**코드목적**
모든 고객을 "구매 이력 있음 / 없음" 두 그룹으로 나눠, 상태 표시와 함께 한 목록으로 보여 줍니다.

**해석**
위 쿼리는 *청구서가 있는 고객*에게 `'Has Invoice'`를, 아래 쿼리는 *청구서가 없는 고객*에게 `'No Invoice'`를 붙입니다. 두 결과를 `UNION`으로 합치면 전체 고객이 상태와 함께 한 표에 정리됩니다. 두 쿼리의 컬럼 수·타입이 같아야 결합되며, 마지막 `ORDER BY`로 전체를 고객 ID 순으로 정렬합니다.

**실무 연결**
"활성/비활성", "결제/미결제", "신규/기존"처럼 *대상을 두 그룹으로 나눠 상태를 붙이는* 작업은 CRM·마케팅 리포트의 단골입니다. 출처/상태를 나타내는 상수 컬럼을 더해 두면, 합친 뒤에도 어느 그룹인지 바로 알 수 있습니다.

## 직접 해보기

1. 직원 이름과 고객 이름을 `UNION`으로 합치고, 출처를 구분하는 상수 컬럼(`'Employee'`/`'Customer'`)을 추가해 보세요.
2. 연도별 매출을 구한 뒤, `UNION ALL`로 맨 아래에 "전체 합계" 행을 붙여 보세요.
3. 2009년과 2010년에 모두 구매한 고객을 `INTERSECT`로 뽑고, 그 결과를 고객 테이블과 조인해 이름까지 출력해 보세요.

## 헷갈리기 쉬운 포인트

- **UNION vs UNION ALL**: `UNION`은 중복 제거(+정렬 동반), `UNION ALL`은 중복 유지(더 빠름). 중복이 없으면 `UNION ALL`이 효율적.
- **INTERSECT vs INNER JOIN**: 결과만 보면 비슷하지만, `INTERSECT`는 결과 *집합*을 다루고 조인은 행을 *옆으로* 잇는다. MySQL처럼 `INTERSECT`가 없으면 조인으로 대체.
- **EXCEPT 앞뒤 순서**: `A EXCEPT B`와 `B EXCEPT A`는 결과가 다르다. "기준이 되는 표"를 앞에 둔다.
- **MINUS vs EXCEPT**: 같은 차집합이지만 Oracle은 `MINUS`, 그 외 다수는 `EXCEPT`.

## 연결되는 개념

- 이전에 알면 좋은 글: [관계형 대수와 STANDARD SQL](01-relational-algebra.md) — 집합 연산의 개념적 출발점입니다.
- 함께 보면 좋은 글: [집계 함수와 GROUP BY / HAVING](02-aggregate-functions.md), [서브쿼리](03-subquery.md)
- 다음에 이어지는 글: [복합 활용 — 통계 리포트 만들기](06-composite-report.md) — 집계와 `UNION ALL`을 합쳐 소계 리포트를 만듭니다.
- 더 찾아볼 키워드: `CROSS JOIN`, `소계(subtotal)`, `이탈 고객(churn)`, `MySQL 집합 연산 대체`

## 셀프 체크

- [ ] 집합 연산자 4가지와 각각의 역할(합·합중복·교·차)을 말할 수 있다.
- [ ] 집합 연산의 전제 조건(컬럼 수·타입 호환)을 설명할 수 있다.
- [ ] `UNION`과 `UNION ALL`의 차이를 안다.
- [ ] `EXCEPT`의 앞뒤 순서가 결과에 미치는 영향을 이해한다.
- [ ] DBMS마다 차집합 키워드가 다를 수 있음을 안다.

**복습 질문 및 답변**

- (기본) 중복을 제거하지 않는 합집합 연산자는 무엇인가요?
  → `UNION ALL`입니다.
- (이해 확인) 집합 연산을 하려면 두 쿼리 사이에 반드시 맞아야 하는 두 가지는 무엇인가요?
  → 컬럼 개수와 (대응 컬럼의) 데이터 타입 호환입니다.
- (응용) "작년엔 샀지만 올해는 안 산 고객"을 뽑으려면 어떤 연산자를, 어떤 순서로 써야 하나요?
  → `(작년 구매 고객) EXCEPT (올해 구매 고객)` 순서로 `EXCEPT`를 씁니다.

## 한 줄 정리

> 집합 연산자는 조인 없이 결과를 세로로 묶는 도구로, `UNION`(합)·`INTERSECT`(교)·`EXCEPT`(차)를 컬럼 수와 타입만 맞으면 자유롭게 조합할 수 있다.
