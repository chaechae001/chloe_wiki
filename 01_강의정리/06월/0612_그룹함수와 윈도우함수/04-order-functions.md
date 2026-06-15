# 행 순서 함수 — 이전·다음 행을 끌어오기 (LAG · LEAD)

> "이번 달 매출이 지난달보다 늘었나?"를 알려면 두 행을 나란히 놓아야 합니다. SQL은 보통 한 행씩만 보는데, **옆 행을 현재 행으로 끌어오는** 함수가 있다면 시계열 분석이 한결 쉬워집니다.

`LAG` `LEAD` `FIRST_VALUE` `LAST_VALUE` `시계열 분석` `전월 대비` `MoM` `재주문 간격` `PARTITION BY`

## 핵심요약

- 행 순서 함수는 정렬된 데이터에서 **다른 행의 값을 현재 행으로 가져온다**.
- **LAG(컬럼, N)**: N칸 **이전** 행의 값. 첫 행은 가져올 게 없어 `NULL`.
- **LEAD(컬럼, N)**: N칸 **이후** 행의 값. 마지막 행은 `NULL`.
- **FIRST_VALUE / LAST_VALUE**: 창 안의 첫 값 / 마지막 값(그룹별 최소·최대 추출에 활용).
- `LAST_VALUE`는 범위를 `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`으로 **명시해야** 의도대로 동작한다.
- 시계열 분석의 핵심은 **`ORDER BY`(시간 순서)**와 **`PARTITION BY`(누구 기준으로 나눌지)**를 정확히 지정하는 것.
- LAG로 직전 값을 가져오면 전월 대비 증감, 주문 간격, 결제 변화 같은 분석이 사칙연산만으로 풀린다.

## 개념별 정리

### LAG — 이전 행 가져오기

**1. 정의**
현재 행 기준으로 N번째 **이전** 행의 값을 가져옵니다. 기본 N은 1(바로 앞).

**2. 왜 필요한가?**
"전월 대비", "전일 대비", "직전 주문" 같은 **과거와의 비교**는 모두 직전 값이 옆 칸에 있어야 계산됩니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    LAG(NAME, 1) OVER (ORDER BY ID) AS PREV_EMPLOYEE_NAME
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
첫 행은 이전 행이 없어 결과가 `NULL`입니다. 증감을 계산할 때 이 `NULL`을 어떻게 처리할지(제외할지, 0으로 볼지) 미리 정해야 합니다.

**5. 한 줄 정리**
한 칸 위 값을 내 행으로 데려온다.

> 비유: 줄 서 있을 때 "바로 앞 사람"을 돌아보는 것. 맨 앞 사람은 돌아봐도 아무도 없다(NULL).

### LEAD — 다음 행 가져오기

**1. 정의**
현재 행 기준으로 N번째 **이후** 행의 값을 가져옵니다.

**2. 왜 필요한가?**
"다음 결제는 얼마였나", "다음 이벤트까지 며칠" 같은 **미래(다음) 값과의 비교**에 씁니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    LEAD(NAME, 1) OVER (ORDER BY ID) AS AFTER_EMPLOYEE_NAME
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
마지막 행은 다음 행이 없어 `NULL`입니다. LAG의 첫 행 NULL과 정확히 대칭입니다.

**5. 한 줄 정리**
한 칸 아래 값을 내 행으로 데려온다.

> 비유: 줄에서 "바로 뒤 사람"을 돌아보는 것. 맨 뒤 사람 뒤에는 아무도 없다(NULL).

### FIRST_VALUE / LAST_VALUE — 창의 처음·마지막 값

**1. 정의**
정렬된 창에서 가장 먼저/가장 나중에 오는 값을 가져옵니다. 그룹별 최소·최대를 행마다 붙일 때 유용합니다.

**2. 왜 필요한가?**
"각 부서에서 가장 적게/많이 받는 급여를 모든 직원 행에 함께 보여 주기"처럼, 그룹의 양 끝값을 비교 기준으로 깔아 둘 때 씁니다.

**3. 예시 — 부서별 최소·최대 급여**

```sql
SELECT
    ID, DEPARTMENT_ID, NAME, SALARY,
    FIRST_VALUE(SALARY) OVER (
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPARTMENT_MIN_SALARY,
    LAST_VALUE(SALARY) OVER (
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPARTMENT_MAX_SALARY
FROM EMPLOYEE
ORDER BY ID;
```

**4. 헷갈리기 쉬운 점**
`LAST_VALUE`는 범위를 명시하지 않으면 기본 창이 "처음 ~ 현재 행"이라, 마지막 값이 아니라 "현재 행까지의 마지막 값"이 나옵니다. 그래서 위 코드처럼 `UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`을 **반드시** 붙여 창 전체를 보게 해야 합니다.

**5. 한 줄 정리**
창의 양 끝값을 가져온다 — 단, LAST_VALUE는 범위 명시 필수.

> 비유: 한 줄로 선 사람들 중 "맨 앞 키"와 "맨 뒤 키"를 모두가 들고 있게 하는 것. 단, "맨 뒤"를 보려면 줄 끝까지 시야를 열어 줘야 한다.

## 코드로 보기 ① — 전월 대비 증감(MoM)을 단계로 쌓기

비즈니스 판매 DB로 "월별 매출의 전월 대비 증감"을 만듭니다. 한 번에 쓰기보다 **단계로 쌓는** 흐름이 이해에 좋습니다.

1단계 — 월별 매출 집계(서브쿼리로 준비):

```sql
SELECT
    strftime('%Y-%m', o.orderDate)          AS YearMonth,
    SUM(od.quantityOrdered * od.priceEach)  AS MonthlyRevenue
FROM orders o
JOIN orderdetails od ON o.orderNumber = od.orderNumber
WHERE o.status = 'Shipped'
GROUP BY strftime('%Y-%m', o.orderDate);
```

2~4단계 — LAG로 직전 월을 가져와 증감액·증감률 계산, 보기 좋게 반올림:

```sql
SELECT
    YearMonth,
    ROUND(MonthlyRevenue, 2)                                   AS MonthlyRevenue,
    ROUND(LAG(MonthlyRevenue, 1) OVER (ORDER BY YearMonth), 2) AS PrevMonthRevenue,
    ROUND(MonthlyRevenue
          - LAG(MonthlyRevenue, 1) OVER (ORDER BY YearMonth), 2) AS MoM_Change,
    ROUND((MonthlyRevenue - LAG(MonthlyRevenue, 1) OVER (ORDER BY YearMonth))
          / LAG(MonthlyRevenue, 1) OVER (ORDER BY YearMonth), 4) AS MoM_Pct
FROM (
    SELECT
        strftime('%Y-%m', o.orderDate)          AS YearMonth,
        SUM(od.quantityOrdered * od.priceEach)  AS MonthlyRevenue
    FROM orders o
    JOIN orderdetails od ON o.orderNumber = od.orderNumber
    WHERE o.status = 'Shipped'
    GROUP BY strftime('%Y-%m', o.orderDate)
) AS monthly
ORDER BY YearMonth;
```

**코드목적**
달마다 매출을 집계한 뒤, 직전 달 매출을 같은 행에 끌어와 "이번 달 − 지난달 = 증감액", "증감액 ÷ 지난달 = 증감률"을 구하는 것이 목적입니다.

**해석**
`MoM_Change`가 양수면 전월 대비 성장, 음수면 감소입니다. 첫 달은 비교할 직전 달이 없어 `PrevMonthRevenue`가 `NULL`이고, 따라서 증감도 계산되지 않습니다. 연말→연초 구간에서 큰 폭의 감소가 보이면 계절성을 의심해 볼 수 있습니다.

**실무 연결**
경영 대시보드에서 가장 많이 쓰는 지표가 바로 전월 대비(MoM)입니다. `strftime`으로 연·월을 묶고 `LAG`로 직전 값을 끌어오는 이 패턴 하나로 전일 대비, 전년 동월 대비 등으로 쉽게 확장됩니다.

## 코드로 보기 ② — 고객 재주문 간격(이탈 위험군 탐지)

`PARTITION BY`로 고객마다 칸막이를 친 뒤, 같은 고객의 직전 주문일을 끌어와 간격을 잽니다.

```sql
SELECT
    c.customerName,
    o.orderDate,
    LAG(o.orderDate, 1) OVER (
        PARTITION BY c.customerNumber
        ORDER BY o.orderDate
    ) AS prevOrderDate,
    julianday(o.orderDate)
        - julianday(LAG(o.orderDate, 1) OVER (
              PARTITION BY c.customerNumber
              ORDER BY o.orderDate
          )) AS DaysSincePrevOrder
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
ORDER BY c.customerName, o.orderDate;
```

**코드목적**
고객별로 주문을 시간순으로 늘어놓고, 직전 주문일과의 날짜 차이를 구해 "이 고객은 평소 며칠마다 주문하는가"를 보는 것이 목적입니다.

**해석**
`julianday()`는 날짜를 숫자(일 단위)로 바꿔 줘 두 날짜의 차이를 뺄셈으로 계산할 수 있게 합니다. 각 고객의 첫 주문은 직전 주문이 없어 간격이 `NULL`입니다. 간격이 유독 긴 고객(예: 500일 이상)은 재주문 주기가 비정상적으로 길어진 **이탈 위험군**으로 볼 수 있습니다.

**실무 연결**
CRM·마케팅에서 이탈(Churn) 징후를 조기에 잡는 핵심 분석입니다. 평소 30일 주기 고객이 180일째 주문이 없다면 리텐션 캠페인 대상이 됩니다. 같은 패턴으로 `LEAD`를 쓰면 "다음 결제까지의 간격"도 구할 수 있습니다.

## 직접 해보기

1. 위 MoM 쿼리에서 `LAG(..., 1)`을 `LAG(..., 2)`로 바꾸면 무엇과의 비교가 될까요? (힌트: 2개월 전)
2. 청구서 데이터에서 `LAG(Total)`·`LEAD(Total)`로 직전·다음 청구금액을 옆에 붙이는 쿼리를 작성해 보세요.
3. 결제가 4회 이상인 단골 고객만 추려, 결제마다 "직전 결제 대비 증감액"을 구해 보세요. (힌트: 서브쿼리로 4회 이상 필터 → LAG)

## 헷갈리기 쉬운 포인트

- **LAG vs LEAD**: 과거 방향이면 LAG(이전), 미래 방향이면 LEAD(다음). 첫 행 NULL은 LAG, 마지막 행 NULL은 LEAD.
- **FIRST_VALUE vs LAST_VALUE**: 둘 다 창의 끝값이지만, LAST_VALUE는 범위를 `UNBOUNDED FOLLOWING`까지 열어 주지 않으면 "현재 행까지의 마지막"만 본다.
- **PARTITION BY 유무**: 빼면 전체를 한 줄로 보고 직전 값을 가져오고, 넣으면 그룹(고객 등)마다 따로 직전 값을 가져온다. 시계열에서 누구 기준인지가 결정적이다.

## 연결되는 개념

- 이전 글: [집계 윈도우 함수와 누적합](03-aggregate-window-functions.md) — 누적은 여러 행을 더하고, 행 순서 함수는 특정 행을 콕 집어 온다.
- 다음 글: [비율과 등급](05-ratio-ntile-functions.md) — 가져온 값들을 비율·등급으로 가공합니다.
- 함께 보면 좋은 글: [순위 함수 3형제](02-ranking-functions.md) — 같은 `OVER(... ORDER BY ...)` 구조.
- 더 찾아볼 키워드: `time series`, `month over month`, `churn analysis`, `julianday`, `strftime`

## 셀프 체크

- [ ] LAG·LEAD의 방향과 NULL이 생기는 위치(첫/마지막 행)를 안다.
- [ ] LAST_VALUE에 범위 명시가 필요한 이유를 설명할 수 있다.
- [ ] `PARTITION BY`로 고객별·그룹별 직전 값을 가져올 수 있다.
- [ ] LAG로 전월 대비 증감액·증감률을 계산할 수 있다.
- [ ] julianday(또는 날짜 함수)로 두 날짜 간격을 구할 수 있다.

**복습 질문 및 답변**

- (기본) 직전 행 값을 가져오는 함수와, 결과가 NULL이 되는 위치는?
  → LAG, 첫 행에서 NULL.
- (이해확인) 부서별 최댓값을 LAST_VALUE로 구할 때 흔히 틀리는 이유는?
  → 범위를 `UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`으로 열지 않으면 "현재 행까지의 마지막 값"만 봐서 진짜 최댓값이 안 나온다.
- (응용) 고객 이탈 위험군을 LAG로 찾는다면 어떤 순서로 쿼리를 짤까?
  → `PARTITION BY 고객 ORDER BY 주문일`로 직전 주문일을 가져오고, julianday 차이로 간격을 구한 뒤, 간격이 큰 고객을 거른다(첫 주문 NULL은 제외).

## 한 줄 정리

> LAG·LEAD는 옆 행을 내 행으로 데려오는 도구이며, `ORDER BY`(시간)와 `PARTITION BY`(기준)만 정확하면 전월 대비·주문 간격 같은 시계열 분석이 사칙연산으로 풀린다.
