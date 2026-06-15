# 계층형 질의 — 부모와 자식으로 얽힌 데이터 풀어내기

> 조직도를 떠올려 보세요. 사장 아래 팀장, 팀장 아래 팀원… 이런 위·아래 관계가 *한 테이블 안에* 들어 있을 때, 그걸 계단처럼 펼쳐 보는 방법이 계층형 질의입니다.

`계층형 질의` `계층형 데이터` `재귀 CTE` `WITH RECURSIVE` `CONNECT BY` `앵커 쿼리` `재귀 쿼리` `LEVEL` `조직도`

## 핵심요약

- **계층형 데이터**는 같은 테이블 안에 *상위와 하위가 함께* 들어 있는 데이터다. (관리자–사원, 카테고리–하위 카테고리 등)
- 보통 각 행이 자신의 "부모"를 가리키는 컬럼(예: `ReportsTo`, `manager_id`)을 가진다. *부모가 NULL인 행이 최상위(Root)* 다.
- 표준에 가까운 방법은 **재귀 CTE**: `WITH RECURSIVE`로 "루트에서 시작해 자식으로 한 단계씩 내려가는" 과정을 반복한다.
- 재귀 CTE는 두 부분으로 나뉜다: **앵커 쿼리**(시작점=루트)와 **재귀 쿼리**(직속 자식 탐색), 둘을 `UNION ALL`로 잇는다.
- DBMS마다 방법이 다르다. Oracle은 `START WITH ... CONNECT BY PRIOR`, SQL Server·MySQL·MariaDB·SQLite는 재귀 CTE를 쓴다.
- **LEVEL(깊이)**, 들여쓰기, 경로(path), 리프(leaf) 여부 등을 함께 출력하면 구조를 보기 쉽다.

## 개념별 정리

### 계층형 데이터란?

**1. 정의**
하나의 테이블 안에서 행과 행이 *상위–하위 관계*로 연결된 데이터입니다. 각 행이 자신의 부모를 가리키는 컬럼을 갖는 "순환관계(self-reference) 모델"로 표현됩니다.

**2. 왜 필요한가?**
조직도(관리자–사원), 댓글–대댓글, 상품 카테고리 트리, 부서 구조처럼 *깊이가 정해지지 않은 트리 구조*를 한 테이블로 저장해야 할 때가 많습니다.

**3. 예시**
직원 테이블에서 각 직원은 자신의 관리자(부모)를 가리킵니다.

| 사원번호 | 직급 | 관리자 |
| --- | --- | --- |
| 1000 | 사장 | (없음) |
| 1001 | 대리 | 1000 |
| 1002 | 대리 | 1000 |
| 1003 | 사원 | 1001 |

`1000`은 관리자가 없으므로 최상위(Root), `1001`·`1002`는 그 아래, `1003`은 또 그 아래입니다. 이를 펼치면 트리 모양이 됩니다.

**4. 헷갈리기 쉬운 점**
"관리자" 컬럼은 같은 테이블의 사원번호를 가리킵니다. 즉 *자기 테이블을 자기가 참조*하는 구조라, 일반 조인과는 사고방식이 조금 다릅니다.

**5. 한 줄 정리**
계층형 데이터는 "한 테이블 안에 트리가 접혀 있는" 데이터다.

> 비유: 가계도를 한 장의 명단으로 적되, 각 사람 옆에 "부모 이름"만 적어 둔 것과 같습니다. 그 부모 정보를 따라가면 가계도가 펼쳐집니다.

### 재귀 CTE — WITH RECURSIVE

**1. 정의**
재귀 CTE(공통 테이블 표현식)는 *자기 자신을 다시 참조*하며 한 단계씩 결과를 쌓아 가는 임시 테이블입니다. `WITH RECURSIVE 이름(...) AS ( 앵커 쿼리 UNION ALL 재귀 쿼리 )` 형태입니다.

**2. 왜 필요한가?**
트리의 깊이가 몇 단계인지 미리 알 수 없을 때, "더 내려갈 자식이 없을 때까지" 자동으로 반복해 모든 계층을 펼치기 위해서입니다.

**3. 예시**

```sql
WITH RECURSIVE emp_hierarchy(
    EmployeeId, FirstName, LastName, Title, ReportsTo, lvl
) AS (
    -- ① 앵커 쿼리: 최상위 직원(Root)에서 시작, 레벨 0
    SELECT EmployeeId, FirstName, LastName, Title, ReportsTo,
           0 AS lvl
    FROM employees
    WHERE ReportsTo IS NULL

    UNION ALL

    -- ② 재귀 쿼리: 직속 부하를 찾아 레벨 +1
    SELECT e.EmployeeId, e.FirstName, e.LastName,
           e.Title, e.ReportsTo, h.lvl + 1
    FROM employees e
    JOIN emp_hierarchy h ON e.ReportsTo = h.EmployeeId
)
SELECT lvl AS 계층레벨,
       EmployeeId AS 직원ID,
       FirstName || ' ' || LastName AS 직원명,
       Title AS 직책,
       ReportsTo AS 관리자ID
FROM emp_hierarchy
ORDER BY lvl, EmployeeId;
```

**4. 헷갈리기 쉬운 점**
앵커와 재귀를 잇는 것은 `UNION`이 아니라 `UNION ALL`입니다. 또 재귀 쿼리는 *CTE 자기 자신*(`emp_hierarchy h`)을 참조해, 직전 단계 결과에 새 자식을 이어 붙입니다.

**5. 한 줄 정리**
재귀 CTE는 "루트에서 시작해 자식이 없을 때까지 자동으로 내려가는" 반복문이다.

> 비유: 양파 껍질을 한 겹씩 까듯, 루트부터 한 단계씩 자식을 펼쳐 가다 더 깔 게 없으면 멈춥니다.

### 재귀가 도는 과정 (한 단계씩 따라가기)

**1. 정의**
재귀 CTE가 결과를 어떻게 쌓아 가는지, 순환(iteration)마다 무엇이 추가되는지를 살펴봅니다.

**2. 왜 필요한가?**
"왜 이런 결과가 나오는지" 과정을 이해하면, 결과가 이상할 때 어디가 잘못됐는지 짚어낼 수 있습니다.

**3. 예시**
다음 데이터(관리자–사원)를 재귀 CTE로 펼친다고 합시다.

| member_id | manager_id |
| --- | --- |
| 1000 | NULL |
| 1001 | 1000 |
| 1002 | 1001 |
| 1003 | 1002 |
| 1004 | 1002 |
| 1005 | 1000 |
| 1006 | 1005 |
| 1007 | 1006 |

순환이 돌 때마다 *그 단계에서 새로 찾은 행*은 다음과 같습니다.

- **1번째 순환(앵커)**: 관리자가 NULL인 `1000` (lvl 0)
- **2번째 순환**: `1000`의 직속 부하 → `1001`, `1005` (lvl 1)
- **3번째 순환**: `1001`·`1005`의 부하 → `1002`, `1006` (lvl 2)
- **4번째 순환**: 그 부하 → `1003`, `1004`, `1007` (lvl 3)
- 더 내려갈 자식이 없으면 종료

최종 결과(정렬 후)는 다음과 같습니다.

| member_id | manager_id | lvl |
| --- | --- | --- |
| 1000 | NULL | 0 |
| 1001 | 1000 | 1 |
| 1002 | 1001 | 2 |
| 1003 | 1002 | 3 |
| 1004 | 1002 | 3 |
| 1005 | 1000 | 1 |
| 1006 | 1005 | 2 |
| 1007 | 1006 | 3 |

**4. 헷갈리기 쉬운 점**
각 순환은 *직전 순환에서 새로 추가된 행*의 자식만 찾습니다. 전체를 매번 다시 훑는 게 아니라, "새로 들어온 부모"를 기준으로 다음 층을 찾아 나갑니다.

**5. 한 줄 정리**
재귀는 "이번에 찾은 사람들의 자식"을 다음 순환에서 찾으며 한 층씩 내려간다.

> 비유: 인맥을 타고 내려가는 것과 같습니다. 먼저 사장을 알고(1층), 사장이 소개한 사람들(2층), 그들이 소개한 사람들(3층)… 새로 만난 사람을 통해서만 다음 사람을 만납니다.

### DBMS별 계층형 질의 방법

**1. 정의**
같은 "트리 펼치기"라도 데이터베이스마다 전용 문법이 다릅니다.

**2. 왜 필요한가?**
Oracle 자료로 공부했는데 실습 환경이 SQLite라면 문법이 안 맞습니다. 어느 환경에서 무엇을 쓰는지 알아 두면 헷갈리지 않습니다.

**3. 예시**

| DBMS | 방법 |
| --- | --- |
| Oracle | `START WITH ... CONNECT BY PRIOR ...` (전용 문법) |
| SQL Server | 2005 이후 재귀 CTE (`WITH`) |
| MySQL / MariaDB | 특정 버전 이후 재귀 CTE (`WITH RECURSIVE`) |
| SQLite | 재귀 CTE (`WITH RECURSIVE`) |

참고로 Oracle 전용 문법은 다음과 같이 생겼습니다. (SQLite에서는 동작하지 않습니다.)

```sql
-- Oracle 계층형 질의 (비교용)
SELECT LEVEL, EmployeeId, FirstName, ReportsTo
FROM   employees
START  WITH ReportsTo IS NULL              -- 루트 지정
CONNECT BY PRIOR EmployeeId = ReportsTo;   -- 상위-하위 연결 방식
```

- `START WITH 부모 IS NULL`: 부모가 없는 행을 루트로 지정
- `CONNECT BY PRIOR 자식 = 부모`: 상위와 하위를 잇는 규칙
- `LEVEL`: 깊이(루트가 1)

**4. 헷갈리기 쉬운 점**
Oracle의 `LEVEL`은 루트가 *1*에서 시작하고, 재귀 CTE 예제에서는 보통 루트를 *0*으로 두는 경우가 많습니다. 시작 숫자가 다를 수 있으니, 레벨 값을 비교할 때 기준을 확인하세요.

**5. 한 줄 정리**
트리 펼치기 개념은 같지만, Oracle은 `CONNECT BY`, 나머지는 대체로 재귀 CTE를 쓴다.

> 비유: "산을 오른다"는 목표는 같아도, 등산로(문법)는 코스마다 다른 것과 같습니다.

### 레벨 · 들여쓰기 · 경로 · 리프

**1. 정의**
계층을 *보기 좋게* 만드는 부가 정보입니다. 깊이(레벨), 깊이에 따른 들여쓰기, 루트부터의 경로, 그리고 더 이상 자식이 없는 끝 노드(리프) 여부 등입니다.

**2. 왜 필요한가?**
단순히 행만 나열하면 누가 누구의 아래인지 알기 어렵습니다. 들여쓰기나 경로를 붙이면 조직도처럼 한눈에 들어옵니다.

**3. 예시**

```sql
-- 레벨에 따라 공백으로 들여쓰기해 조직도처럼 보이게
SELECT lvl AS 계층레벨,
       SUBSTR('                ', 1, lvl * 4)
           || FirstName || ' ' || LastName AS 직원명,
       Title AS 직책
FROM emp_hierarchy
ORDER BY lvl, EmployeeId;
```

`SUBSTR('     ...', 1, lvl * 4)`는 레벨이 깊을수록 더 많은 공백을 앞에 붙여 들여쓰기 효과를 냅니다. (Oracle에서는 같은 목적으로 `LPAD(' ', 4*(LEVEL-1))`를 씁니다.)

**4. 헷갈리기 쉬운 점**
들여쓰기는 *보기용 장식*일 뿐, 데이터 자체를 바꾸지 않습니다. 화면이나 환경에 따라 공백이 의도대로 안 보일 수도 있습니다.

**5. 한 줄 정리**
레벨·들여쓰기·경로는 트리를 "사람이 읽기 좋게" 꾸미는 장치다.

> 비유: 목차에서 대제목·소제목을 들여쓰기로 구분해 한눈에 위계를 보여 주는 것과 같습니다.

## 코드로 보기 — 직원별 담당 고객 수까지 더한 계층 조회

```sql
WITH RECURSIVE emp_hier(
    EmployeeId, FirstName, LastName, Title, ReportsTo, lvl
) AS (
    SELECT EmployeeId, FirstName, LastName, Title, ReportsTo, 0 AS lvl
    FROM employees
    WHERE ReportsTo IS NULL
    UNION ALL
    SELECT e.EmployeeId, e.FirstName, e.LastName,
           e.Title, e.ReportsTo, h.lvl + 1
    FROM employees e
    JOIN emp_hier h ON e.ReportsTo = h.EmployeeId
)
SELECT h.lvl                              AS 계층레벨,
       h.FirstName || ' ' || h.LastName   AS 직원명,
       h.Title                            AS 직책,
       COUNT(c.CustomerId)                AS 담당고객수,
       ROUND(SUM(i.Total), 2)             AS 담당매출합계
FROM emp_hier h
LEFT JOIN customers c ON h.EmployeeId = c.SupportRepId
LEFT JOIN invoices  i ON c.CustomerId = i.CustomerId
GROUP BY h.EmployeeId, h.lvl, h.FirstName, h.LastName, h.Title
ORDER BY h.lvl, h.EmployeeId;
```

**코드목적**
조직 계층을 펼치는 데서 멈추지 않고, *각 직원이 담당하는 고객 수와 매출까지* 함께 보여 주는 실무형 리포트입니다.

**해석**
먼저 재귀 CTE(`emp_hier`)로 직원 계층을 레벨과 함께 펼칩니다. 그다음 그 결과를 고객(담당 직원 = `SupportRepId`)·청구서 테이블과 `LEFT JOIN`해, 직원마다 담당 고객 수(`COUNT`)와 매출 합계(`SUM`)를 집계합니다. `LEFT JOIN`을 쓴 이유는, 담당 고객이 없는 직원도 0으로 표시해 빠뜨리지 않기 위해서입니다. 결과는 계층 레벨 순으로, 직원별 담당 실적이 한 표에 정리됩니다.

**실무 연결**
"조직 구조 + 각 구성원의 실적"을 한 번에 보는 리포트는 영업 관리·인사 분석의 핵심입니다. 계층형 질의(구조)와 집계·조인(실적)을 결합하면, 조직도와 성과표를 따로 만들지 않고 한 쿼리로 뽑을 수 있습니다.

## 직접 해보기

1. 재귀 CTE의 두 부분이 각각 무슨 역할을 하는지(앵커 / 재귀) 한 문장씩 적어 보세요.
2. 직원 보고 체계를 펼친 뒤, 레벨에 따라 공백으로 들여쓰기해 조직도처럼 출력해 보세요.
3. 루트부터 각 직원까지의 경로(예: `사장 > 팀장 > 팀원`)를 만들어, 끝 노드(리프)인지 아닌지 표시해 보세요.

## 헷갈리기 쉬운 포인트

- **앵커 쿼리 vs 재귀 쿼리**: 앵커는 시작점(루트)을 한 번 만들고, 재귀는 직속 자식을 *반복해서* 찾아 붙인다. 둘은 `UNION ALL`로 잇는다.
- **CONNECT BY vs WITH RECURSIVE**: 같은 트리 펼치기지만, 전자는 Oracle 전용, 후자는 표준에 가까운 방식.
- **LEVEL 시작값**: Oracle `LEVEL`은 루트가 1, 재귀 CTE 예제는 루트를 0으로 두는 경우가 많다.
- **UNION vs UNION ALL (재귀에서)**: 재귀 CTE의 두 부분은 반드시 `UNION ALL`로 잇는다.

## 연결되는 개념

- 이전에 알면 좋은 글: [집합 연산자](04-set-operators.md) — 재귀 CTE의 `UNION ALL`이 여기서 등장합니다.
- 함께 보면 좋은 글: [서브쿼리](03-subquery.md) (CTE는 이름 붙은 임시 결과라는 점에서 인라인 뷰와 통합니다.)
- 다음에 이어지는 글: [복합 활용 — 통계 리포트 만들기](06-composite-report.md)
- 더 찾아볼 키워드: `self join`, `트리 구조`, `재귀(recursion)`, `CONNECT_BY_ROOT`, `SYS_CONNECT_BY_PATH`

## 셀프 체크

- [ ] 계층형 데이터가 무엇이고, 루트를 어떻게 찾는지 설명할 수 있다.
- [ ] 재귀 CTE의 앵커/재귀 두 부분의 역할을 말할 수 있다.
- [ ] 재귀가 순환마다 어떻게 한 층씩 내려가는지 설명할 수 있다.
- [ ] DBMS마다 계층형 질의 방법이 다를 수 있음을 안다.
- [ ] 레벨·들여쓰기·경로가 무엇을 위한 것인지 안다.

**복습 질문 및 답변**

- (기본) 계층형 데이터에서 "최상위(Root)"는 어떻게 알아보나요?
  → 부모를 가리키는 컬럼(예: `ReportsTo`)이 NULL인 행이 루트입니다.
- (이해 확인) 재귀 CTE에서 앵커 쿼리와 재귀 쿼리를 잇는 연산자는 무엇인가요?
  → `UNION ALL`입니다.
- (응용) Oracle에서 작성한 `CONNECT BY` 쿼리를 SQLite에서 그대로 실행하면 어떻게 되나요?
  → 동작하지 않습니다. SQLite에서는 재귀 CTE(`WITH RECURSIVE`)로 다시 작성해야 합니다.

## 한 줄 정리

> 계층형 질의는 한 테이블에 접혀 있는 트리를 펼치는 기법으로, 표준에 가까운 재귀 CTE는 "루트에서 시작(앵커)해 자식을 반복 탐색(재귀)"하며 모든 계층을 자동으로 끌어낸다.
