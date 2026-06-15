# 그룹 함수 — 소계·총계를 한 번에 (ROLLUP · CUBE · GROUPING SETS)

> 부서별·직무별 평균을 구했더니, 위에서 "부서 소계는?", "전체 평균은?"이라고 묻습니다. 쿼리를 세 번 짜서 `UNION` 할 수도 있지만… 한 번에 끝내는 방법이 있습니다.

`그룹 함수` `ROLLUP` `CUBE` `GROUPING SETS` `소계` `총계` `다차원 집계` `UNION ALL`

## 핵심요약

- 그룹 함수는 `GROUP BY` 결과에 **소계·중계·총계**를 자동으로 덧붙이는 도구다.
- **ROLLUP(A, B)**: `(A,B)` 상세 + `(A)` 소계 + 전체 총계까지 **계층적**으로 만든다.
- **CUBE(A, B)**: ROLLUP에 더해 `(B)`만의 소계까지 — **가능한 모든 조합**을 만든다.
- **GROUPING SETS(A, B)**: 지정한 컬럼들 각각의 통계만 골라서 만든다(상세·총계 없이).
- 소계·총계 행에서 그룹 컬럼은 `NULL`로 표시된다("이 컬럼 기준으로는 묶지 않았다"는 뜻).
- 일부 DB(SQLite 등)는 이 함수들을 직접 지원하지 않아 **`UNION ALL`/`UNION`로 같은 결과를 조립**한다.
- DBMS마다 문법이 달라(`ROLLUP(...)` vs `... WITH ROLLUP`) 환경 확인이 필요하다.

## 개념별 정리

### 그룹 함수란?

**1. 정의**
전체 통계만이 아니라 "데이터 일부에 대한 소계·중계"까지 함께 만들어 주는 함수들입니다. ROLLUP·CUBE·GROUPING SETS가 대표적입니다.

**2. 왜 필요한가?**
"개발팀 과장 평균, 개발팀 전체 평균, 회사 전체 평균"을 한 표에서 보고 싶을 때, 매번 따로 쿼리를 짜 `UNION`하면 길고 번거롭습니다. 그룹 함수는 이를 한 문장으로 줄여 줍니다.

**3. 예시 — 기준이 되는 GROUP BY**

```sql
SELECT
    D.NAME AS DEPARTMENT_NAME,
    J.NAME AS JOB_NAME,
    AVG(E.SALARY) AS AVG_SALARY
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID
JOIN JOB J        ON E.JOB_ID        = J.ID
GROUP BY D.NAME, J.NAME
ORDER BY D.NAME, J.NAME;
```

이 쿼리는 (부서, 직무) 조합별 평균만 줍니다. 소계·총계는 없습니다.

**4. 헷갈리기 쉬운 점**
그룹 함수는 윈도우 함수와 반대입니다. 윈도우 함수는 행을 유지하고, 그룹 함수는 `GROUP BY`처럼 행을 **압축**하되 소계·총계 행을 **추가**합니다.

**5. 한 줄 정리**
GROUP BY에 소계·총계 행을 자동으로 얹어 주는 함수들.

> 비유: 가계부 합산. 항목별 합 아래에 "식비 소계", 맨 아래에 "이달 총지출"을 자동으로 달아 주는 것.

### ROLLUP — 계층적 소계

**1. 정의**
지정한 컬럼을 **왼쪽부터 단계적으로 풀며** 소계와 총계를 만듭니다. `ROLLUP(A, B)`는 `(A,B)` → `(A)` → `()`(전체) 세 단계를 생성합니다.

**2. 왜 필요한가?**
"부서·직무 상세 → 부서 소계 → 전체 총계"처럼 **위로 굴러 올라가는(roll up)** 계층 요약이 필요할 때 씁니다.

**3. 예시**

```sql
-- (ROLLUP 제공 DB)
... GROUP BY ROLLUP(D.NAME, J.NAME);

-- (MariaDB 계열 문법)
... GROUP BY D.NAME, J.NAME WITH ROLLUP;
```

결과(요약):

| DEPARTMENT_NAME | JOB_NAME | AVG_SALARY |
| --------------- | -------- | ---------- |
| 개발 | 과장 | 2500 |
| … | … | … |
| 개발 | (null) | 4083.3333 | ← 개발팀 소계 |
| 영업 | (null) | 5060 | ← 영업팀 소계 |
| (null) | (null) | 4527.2727 | ← 전체 총계 |

**4. 헷갈리기 쉬운 점**
컬럼 순서가 중요합니다. `ROLLUP(부서, 직무)`는 "부서 소계"를 만들지만 "직무만의 소계"는 만들지 않습니다. 직무 소계까지 필요하면 CUBE가 답입니다.

**5. 한 줄 정리**
왼쪽 컬럼부터 풀며 소계 → 총계로 굴러 올라간다.

> 비유: 영수증 정리. 품목별 → 카테고리 소계 → 전체 합계 순으로 위로 합쳐 올라간다.

### CUBE — 모든 조합

**1. 정의**
ROLLUP이 만드는 결과에 더해, 그룹 컬럼들의 **결합 가능한 모든 경우의 수**(다차원)를 집계합니다. `CUBE(A, B)`는 `(A,B)`, `(A)`, `(B)`, `()`를 모두 만듭니다.

**2. 왜 필요한가?**
"부서별 소계"뿐 아니라 "직무별 소계"도 동시에 보고 싶을 때, 즉 **모든 축으로 잘라 보고 싶을 때** 씁니다.

**3. 예시**

```sql
... GROUP BY CUBE(D.NAME, J.NAME);
```

결과에는 ROLLUP의 모든 행에 더해 `(null, 과장 3750)`, `(null, 팀장 9000)`처럼 **직무만의 소계**가 추가됩니다.

**4. 헷갈리기 쉬운 점**
컬럼이 늘수록 조합이 기하급수적으로 늘어 행이 폭증합니다(2개면 4종, 3개면 8종의 집계 레벨). 필요한 조합만 원하면 GROUPING SETS가 더 경제적입니다.

**5. 한 줄 정리**
모든 축으로 잘라 본 다차원 소계·총계.

> 비유: 큐브(주사위)를 모든 면에서 들여다보는 것. 부서 면, 직무 면, 그리고 전체까지.

### GROUPING SETS — 원하는 조합만

**1. 정의**
명시한 컬럼들 **각각의 통계만** 골라 만듭니다. `GROUPING SETS(A, B)`는 `(A)`별 통계와 `(B)`별 통계를 합친 것과 같습니다.

**2. 왜 필요한가?**
ROLLUP/CUBE는 상세·총계까지 다 만들지만, "부서별 통계 + 직무별 통계, 딱 이 둘만" 필요할 때 군더더기 없이 뽑을 수 있습니다.

**3. 예시**

```sql
... GROUP BY GROUPING SETS(D.NAME, J.NAME);
```

결과는 `각 컬럼별 GROUP BY를 UNION ALL`한 것과 동일합니다.

**4. 헷갈리기 쉬운 점**
GROUPING SETS는 상세 조합 `(A,B)`나 전체 총계를 **자동으로 넣지 않습니다**. 필요하면 세트 목록에 직접 추가해야 합니다.

**5. 한 줄 정리**
필요한 그룹 통계만 골라 모은다.

> 비유: 뷔페에서 원하는 접시(부서별·직무별)만 골라 담는 것. 전체 코스를 다 받지 않는다.

## 코드로 보기 — 미지원 DB에서 UNION ALL로 ROLLUP 흉내 내기

SQLite처럼 ROLLUP을 직접 지원하지 않는 환경에서는, 각 집계 레벨을 따로 구해 `UNION ALL`로 이어 붙입니다. 음악 판매 DB에서 "장르·미디어타입 상세 + 장르 소계 + 전체 총계"를 만들어 봅니다.

```sql
-- (1) 상세: (장르, 미디어타입) 조합
SELECT g.Name AS Genre, mt.Name AS MediaType,
       COUNT(t.TrackId) AS TrackCount, ROUND(AVG(t.UnitPrice), 2) AS AvgPrice
FROM   tracks t
JOIN   genres g       ON t.GenreId     = g.GenreId
JOIN   media_types mt ON t.MediaTypeId = mt.MediaTypeId
GROUP BY g.Name, mt.Name

UNION ALL

-- (2) 장르별 소계: MediaType = NULL
SELECT g.Name AS Genre, NULL AS MediaType,
       COUNT(t.TrackId), ROUND(AVG(t.UnitPrice), 2)
FROM   tracks t
JOIN   genres g ON t.GenreId = g.GenreId
GROUP BY g.Name

UNION ALL

-- (3) 전체 총계: Genre = NULL, MediaType = NULL
SELECT NULL AS Genre, NULL AS MediaType,
       COUNT(TrackId), ROUND(AVG(UnitPrice), 2)
FROM   tracks

ORDER BY Genre, MediaType;
```

**코드목적**
ROLLUP이 한 줄로 해 주는 일(상세 → 소계 → 총계)을, 지원하지 않는 DB에서 세 개의 SELECT를 이어 붙여 동일하게 만드는 것이 목적입니다.

**해석**
(1)은 상세 조합, (2)는 `MediaType`을 `NULL`로 둔 장르 소계, (3)은 두 컬럼 모두 `NULL`인 전체 총계입니다. ROLLUP의 출력에서 소계·총계 행이 `NULL`로 표시되던 것과 정확히 같은 구조를 손으로 재현한 셈입니다. CUBE를 흉내 내려면 여기에 "미디어타입만의 소계"를 한 블록 더 `UNION` 하면 되고, GROUPING SETS는 상세·총계 없이 (장르별)·(미디어타입별)만 `UNION ALL` 하면 됩니다.

**실무 연결**
대시보드에서 "카테고리별 + 카테고리 소계 + 전체"를 한 표로 보여 주는 요청은 흔합니다. 사용하는 DB가 ROLLUP을 지원하면 한 줄로, 아니면 이 `UNION ALL` 패턴으로 동일한 리포트를 만들 수 있다는 걸 알면 환경에 휘둘리지 않습니다.

## 직접 해보기

1. 위 ROLLUP 흉내 쿼리에 "미디어타입별 소계" 블록을 추가해 **CUBE**를 흉내 내 보세요.
2. 상세·총계를 빼고 (장르별)·(미디어타입별) 통계만 남겨 **GROUPING SETS**를 흉내 내 보세요.
3. ROLLUP을 지원하는 DB라면 `GROUP BY ROLLUP(...)` 한 줄로 (1)과 같은 결과가 나오는지 비교해 보세요.

## 헷갈리기 쉬운 포인트

- **ROLLUP vs CUBE**: ROLLUP은 "왼쪽부터 계층 소계"(부서 소계 O, 직무 소계 X), CUBE는 "모든 축의 소계"(둘 다 O). 조합 수: ROLLUP n+1종, CUBE 2ⁿ종.
- **CUBE vs GROUPING SETS**: CUBE는 모든 조합 자동 생성, GROUPING SETS는 지정한 조합만. "다 볼지(CUBE) 골라 볼지(GROUPING SETS)".
- **NULL의 두 의미**: 소계·총계 행의 `NULL`은 "이 컬럼으로 안 묶음"이라는 뜻이지, 실제 빈 값이 아니다. 진짜 데이터 NULL과 구분해야 한다.

## 연결되는 개념

- 이전 글: [비율과 등급](05-ratio-ntile-functions.md) — 행을 유지하던 윈도우 함수와 달리, 그룹 함수는 행을 압축한다.
- 함께 보면 좋은 글: [윈도우 함수 첫걸음](01-window-function-basics.md) — "행 유지 vs 행 압축"의 대비를 다시 확인.
- 더 찾아볼 키워드: `GROUPING()` 함수, `subtotal`, `grand total`, `multidimensional aggregation`, `UNION vs UNION ALL`

## 셀프 체크

- [ ] ROLLUP·CUBE·GROUPING SETS가 만드는 집계 레벨의 차이를 설명할 수 있다.
- [ ] 소계·총계 행의 `NULL`이 무슨 뜻인지 안다.
- [ ] 미지원 DB에서 `UNION ALL`로 ROLLUP을 흉내 낼 수 있다.
- [ ] CUBE가 ROLLUP보다 행이 많아지는 이유를 안다.
- [ ] DBMS별 문법 차이(`ROLLUP(...)` vs `WITH ROLLUP`)가 있음을 안다.

**복습 질문 및 답변**

- (기본) (A,B) 상세 + A 소계 + 전체 총계만 만드는 함수는?
  → ROLLUP(A, B).
- (이해확인) CUBE가 ROLLUP과 결정적으로 다른 점은?
  → ROLLUP은 왼쪽 컬럼 기준 소계만, CUBE는 B만의 소계까지 포함해 모든 조합을 만든다.
- (응용) SQLite에서 "부서별 + 직무별 통계"만 필요할 때 어떻게 구현할까?
  → 부서별 GROUP BY와 직무별 GROUP BY를 각각 짜서 `UNION ALL`로 잇는다(= GROUPING SETS와 동일).

## 한 줄 정리

> 그룹 함수는 GROUP BY 결과에 소계·총계를 자동으로 얹어 주며, 계층 요약은 ROLLUP, 모든 조합은 CUBE, 원하는 조합만은 GROUPING SETS — 미지원 DB에선 UNION ALL로 같은 결과를 조립한다.
