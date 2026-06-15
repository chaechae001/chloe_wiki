# 비율과 등급 — RATIO_TO_REPORT · PERCENT_RANK · CUME_DIST · NTILE

> "이 고객은 전체의 몇 %를 차지하나?", "상위 몇 % 안에 드나?", "VIP·일반·주의 중 어디인가?" 순위만으로는 답하기 애매한 질문들을, 비율·등급 함수가 깔끔하게 정리해 줍니다.

`RATIO_TO_REPORT` `PERCENT_RANK` `CUME_DIST` `NTILE` `비율 함수` `백분율` `등급 분류` `PARTITION BY`

## 핵심요약

- **RATIO_TO_REPORT**: 파티션 전체 합(SUM)에서 각 행이 차지하는 **비율**.
- **PERCENT_RANK**: 순위를 백분율로 — 최고 순위는 0, 최저 순위는 1 (범위 0~1).
- **CUME_DIST**: 현재 행보다 같거나 낮은(또는 정렬 방향에 따라 같거나 높은) 행들의 **누적 백분율** (범위 0 초과 ~ 1).
- **NTILE(N)**: 정렬된 행들을 **N등분한 그룹 번호**(1~N)로 나눈다.
- 일부 DB(MariaDB·SQLite 등)는 `RATIO_TO_REPORT`를 제공하지 않으므로 `값 / SUM(값) OVER()`로 직접 구현한다.
- `PARTITION BY`를 더하면 "전체 기준 비율·등급"이 "그룹별 기준"으로 바뀐다.
- 비율·등급 함수는 여신 정책, 상위 고객 식별, 데이터 분위 분석 등에 직접 쓰인다.

## 개념별 정리

### RATIO_TO_REPORT — 전체 합 대비 비율

**1. 정의**
파티션 내 전체 `SUM`을 분모로, 각 행의 값이 차지하는 비율을 구합니다.

**2. 왜 필요한가?**
"각 직원 급여가 전체 인건비의 몇 %인가", "각 국가가 전체 매출의 몇 %인가" 같은 **비중 분석**에 쓰입니다.

**3. 예시 — 제공 DB / 미제공 DB**

```sql
-- (RATIO_TO_REPORT 제공 DB)
SELECT
    ID, NAME, SALARY,
    SUM(SALARY) OVER ()          AS TOTAL_SALARY,
    RATIO_TO_REPORT(SALARY) OVER () AS RATIO
FROM EMPLOYEE;

-- (미제공 DB: 직접 구현 — 값 / 전체합)
SELECT
    ID, NAME, SALARY,
    SUM(SALARY) OVER ()             AS TOTAL_SALARY,
    (SALARY / SUM(SALARY) OVER ())  AS RATIO
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
함수 이름이 DB마다 다를 수 있습니다. `RATIO_TO_REPORT`가 없으면 `값 / SUM(값) OVER()`라는 **같은 의미의 식**으로 대체하면 됩니다. 결과는 동일합니다.

**5. 한 줄 정리**
내 값 ÷ 전체 합 = 내 비중.

> 비유: 피자 한 판에서 내가 먹은 조각의 비율. 분모는 항상 피자 한 판 전체.

### PERCENT_RANK — 순위의 백분율

**1. 정의**
순위를 0~1 사이 백분율로 표현합니다. 가장 높은 순위 행은 0, 가장 낮은 순위 행은 1입니다.

**2. 왜 필요한가?**
데이터 개수가 달라도 "상위 몇 % 수준인가"를 일관되게 비교할 수 있습니다(상대적 위치).

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    PERCENT_RANK() OVER (ORDER BY SALARY DESC) AS PERCENT_RANK
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
"백분율"이라 100을 떠올리기 쉽지만 값은 **0~1**입니다. 또 최고 순위가 1이 아니라 **0**이라는 점이 직관과 반대입니다.

**5. 한 줄 정리**
1등이 0, 꼴찌가 1 — 순위의 상대적 위치를 0~1로.

> 비유: 마라톤에서 "선두는 0% 지점, 최후미는 100% 지점"으로 위치를 표시하는 것.

### CUME_DIST — 누적 분포

**1. 정의**
현재 행을 포함해, 정렬 기준에서 같거나 그 이하(이상)인 행들이 전체에서 차지하는 누적 백분율입니다.

**2. 왜 필요한가?**
"이 값 이하인 데이터가 전체의 몇 %인가" 같은 **분위(percentile)** 감각이 필요할 때 씁니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    ROUND(CUME_DIST() OVER (ORDER BY SALARY DESC), 4) AS CUME_DIST
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
PERCENT_RANK는 **0 이상 1 이하**, CUME_DIST는 **0 초과 1 이하**입니다. CUME_DIST는 현재 행을 포함해 세므로 0이 나오지 않습니다.

**5. 한 줄 정리**
"나 이하가 전체의 몇 %"를 누적으로.

> 비유: 시험 백분위. "내 점수 이하가 상위 60%"처럼 누적된 위치를 알려 준다.

### NTILE(N) — N등분 등급

**1. 정의**
정렬된 행들을 가능한 한 균등하게 N개의 그룹으로 나누고, 각 행에 그룹 번호(1~N)를 붙입니다.

**2. 왜 필요한가?**
"신용한도 상·중·하 3등급", "구매액 상위 4분위" 같은 **자동 등급 분류**에 딱 맞습니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    NTILE(3) OVER (ORDER BY SALARY DESC) AS NTILE
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
행 수가 N으로 딱 나눠떨어지지 않으면 앞쪽 그룹에 한 명씩 더 들어갑니다(균등 분배 규칙). "정확히 똑같은 인원"을 보장하지는 않습니다.

**5. 한 줄 정리**
줄 세운 뒤 N등분해 등급 번호를 붙인다.

> 비유: 달리기 결과로 학생들을 "상·중·하 세 반"에 나눠 배정하는 것.

## 코드로 보기 — 순위 백분율과 누적 분포 함께 보기

```sql
SELECT
    ID, NAME, SALARY,
    PERCENT_RANK()               OVER (ORDER BY SALARY DESC)     AS PERCENT_RANK,
    ROUND(CUME_DIST()            OVER (ORDER BY SALARY DESC), 4) AS CUME_DIST
FROM EMPLOYEE;
```

결과:

| ID | NAME | SALARY | PERCENT_RANK | CUME_DIST |
| -- | ---- | ------ | ------------ | --------- |
| 1004 | ALICE | 10000 | 0 | 0.1667 |
| 1000 | JAMES | 8000 | 0.2 | 0.3333 |
| 1001 | JESSICA | 6000 | 0.4 | 0.6667 |
| 1005 | JANNET | 6000 | 0.4 | 0.6667 |
| 1002 | STEVE | 4000 | 0.8 | 0.8333 |
| 1003 | ROBERT | 1500 | 1 | 1 |

**코드목적**
급여 내림차순에서 각 직원의 상대적 위치를 두 가지 백분율 지표로 동시에 보는 것이 목적입니다.

**해석**
최고 급여인 ALICE의 PERCENT_RANK는 0(가장 앞), 최저인 ROBERT는 1(가장 뒤)입니다. CUME_DIST는 0이 나오지 않고 최솟값이 0.1667(=1/6)인데, 현재 행을 포함해 누적하기 때문입니다. 동점인 JESSICA·JANNET은 두 지표 모두 같은 값을 갖습니다.

**실무 연결**
"상위 10% 우수 고객에게만 혜택"처럼 **상대적 컷오프**를 잡을 때 PERCENT_RANK/CUME_DIST가 쓰입니다. NTILE은 등급을 라벨로 떨어뜨려 주므로, 등급별로 다른 정책(여신 한도, 쿠폰)을 적용하는 자동화에 바로 연결됩니다.

## 응용 — 국가별 신용한도 3등급 분류 (NTILE + PARTITION BY)

비즈니스 판매 DB에서 "각 고객이 자기 나라 안에서 어느 등급인지"를 봅니다. 전체가 아니라 **나라별** 기준이라는 점이 핵심입니다.

```sql
SELECT
    customerName,
    country,
    creditLimit,
    AVG(creditLimit) OVER (PARTITION BY country)                       AS CountryAvgCredit,
    NTILE(3) OVER (PARTITION BY country ORDER BY creditLimit DESC)     AS CreditTier
FROM customers;
```

`PARTITION BY country`로 나라별 칸막이를 친 뒤 `NTILE(3)`으로 상(1)·중(2)·하(3) 등급을 매기고, 같은 행에 그 나라의 평균 신용한도까지 붙여 비교할 수 있게 했습니다. 여신 정책을 나라별로 다르게 적용하는 분석에 그대로 쓸 수 있습니다.

## 직접 해보기

1. `RATIO_TO_REPORT(SALARY) OVER ()`를 함수 없이 `SALARY / SUM(SALARY) OVER ()`로 바꿔 같은 결과가 나오는지 확인해 보세요.
2. `NTILE(3)`을 `NTILE(4)`로 바꾸면 등급이 어떻게 재배치되는지 살펴보세요.
3. 상위 고객 10명에 대해 PERCENT_RANK와 CUME_DIST를 함께 구하고, 두 값이 다르게 나오는 첫 지점을 찾아보세요.

## 헷갈리기 쉬운 포인트

- **PERCENT_RANK vs CUME_DIST**: 전자는 0 이상 1 이하(최고=0), 후자는 0 초과 1 이하(현재 행 포함 누적). "순위의 위치(PERCENT_RANK) vs 누적 분포(CUME_DIST)".
- **RATIO_TO_REPORT vs NTILE**: 전자는 연속적인 "비율(%)", 후자는 이산적인 "등급 번호(1~N)". 비중을 볼지 등급을 나눌지로 선택.
- **전체 기준 vs 그룹 기준**: `PARTITION BY`가 없으면 전체 한 묶음, 있으면 그룹마다 비율·등급을 새로 계산.

## 연결되는 개념

- 이전 글: [행 순서 함수 — LAG · LEAD](04-order-functions.md) — 가져온 값을 비율·등급으로 가공.
- 이전 글: [순위 함수 3형제](02-ranking-functions.md) — 정수 순위를 백분율·등급으로 바꾼 버전이 이 글의 함수들.
- 다음 글: [그룹 함수 — 소계·총계를 한 번에](06-group-functions.md) — 이제 행을 유지하지 않고 소계·총계를 쌓아 봅니다.
- 더 찾아볼 키워드: `percentile`, `quartile`, `decile`, `share of total`, `customer segmentation`

## 셀프 체크

- [ ] RATIO_TO_REPORT를 `값 / SUM(값) OVER()`로 직접 구현할 수 있다.
- [ ] PERCENT_RANK에서 최고 순위가 0이라는 점을 안다.
- [ ] PERCENT_RANK(0~1)와 CUME_DIST(0 초과~1)의 범위 차이를 설명할 수 있다.
- [ ] NTILE(N)으로 데이터를 N등급으로 나눌 수 있다.
- [ ] `PARTITION BY`로 그룹별 비율·등급을 계산할 수 있다.

**복습 질문 및 답변**

- (기본) 데이터를 상·중·하 3등급으로 자동 분류하는 함수는?
  → NTILE(3).
- (이해확인) PERCENT_RANK와 CUME_DIST의 범위가 어떻게 다른가?
  → PERCENT_RANK는 0 이상 1 이하(최고=0), CUME_DIST는 0 초과 1 이하(현재 행 포함 누적이라 0이 안 나옴).
- (응용) "나라별로 신용한도 상위 1/3 고객"만 뽑으려면?
  → `NTILE(3) OVER (PARTITION BY country ORDER BY creditLimit DESC)`로 등급을 만든 뒤, 인라인 뷰로 감싸 `WHERE CreditTier = 1`로 거른다.

## 한 줄 정리

> 비율·등급 함수는 "전체에서 차지하는 몫(RATIO)·상대 위치(PERCENT_RANK·CUME_DIST)·자동 등급(NTILE)"을 한 줄로 계산해 주며, `PARTITION BY`로 전체 기준을 그룹 기준으로 바꿀 수 있다.
