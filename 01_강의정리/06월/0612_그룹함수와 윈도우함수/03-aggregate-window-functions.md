# 집계 윈도우 함수와 누적합 — SUM/AVG OVER & 러닝 토탈

> "올해 누적 매출이 목표의 몇 %일까?" 월별 합계만으로는 답이 안 나옵니다. 각 달 옆에 **그 달까지 쌓인 누적값**을 붙여야 하죠. 윈도우의 '범위' 손잡이가 등장할 차례입니다.

`SUM OVER` `AVG OVER` `누적합` `러닝 토탈` `ROWS BETWEEN` `UNBOUNDED PRECEDING` `CURRENT ROW` `WINDOWING`

## 핵심요약

- 집계 함수(SUM·AVG·MAX·MIN)는 `OVER`와 함께 쓰면 **GROUP BY 없이** 행을 유지한 채 계산된다.
- `OVER()`(빈 괄호)는 전체를 한 묶음으로 보아 **전체 합/평균**을 모든 행에 붙인다.
- `OVER(PARTITION BY ...)`는 그룹별 합/평균을 붙인다.
- 누적합(러닝 토탈)은 `OVER(ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`로 만든다.
- 범위(`ROWS BETWEEN ...`)를 바꾸면 같은 SUM이라도 "앞으로 쌓기"·"뒤로 쌓기"처럼 결과가 달라진다.
- 윈도우 함수 결과는 **다른 계산의 재료**로 쓸 수 있다(예: 값 ÷ 전체합 = 비율).
- 단, `GROUP BY`와 `SUM() OVER()`를 한 쿼리에 섞으면 의도와 다른 분모가 나올 수 있어 주의가 필요하다.

## 개념별 정리

### GROUP BY 없는 집계 — SUM/AVG OVER

**1. 정의**
일반 집계 함수에 `OVER`를 붙이면, 행을 합치지 않고도 합계·평균을 계산해 각 행에 덧붙일 수 있습니다.

**2. 왜 필요한가?**
"각 직원 급여 + 전체 급여 합" 또는 "각 직원 급여 + 소속 부서 평균"처럼 **개별값과 전체(또는 그룹)값을 한 줄에서 비교**하고 싶을 때 필요합니다. 서브쿼리보다 짧습니다.

**3. 예시**

```sql
SELECT
    ID, NAME, SALARY,
    SUM(SALARY) OVER ()                         AS TOTAL_SALARY,   -- 전체 합
    AVG(SALARY) OVER (PARTITION BY DEPARTMENT_ID) AS DEPT_AVG       -- 부서 평균
FROM EMPLOYEE;
```

**4. 헷갈리기 쉬운 점**
`OVER()`의 빈 괄호를 빼먹으면 일반 집계 함수가 되어 `GROUP BY`를 요구하는 에러가 납니다. 윈도우로 쓰겠다는 신호가 바로 `OVER`입니다.

**5. 한 줄 정리**
`집계함수() OVER(...)` — 행을 살린 채 합·평균을 옆에 붙인다.

> 비유: 시험지마다 "내 점수 / 반 평균"을 함께 인쇄해 주는 것. 점수표(원본 행)는 그대로다.

### 누적합(러닝 토탈) — ROWS BETWEEN

**1. 정의**
`ORDER BY`로 정렬한 뒤, 첫 행부터 현재 행까지를 더해 가는 합계입니다. "달이 쌓일수록 커지는 누적 매출"이 대표적입니다.

**2. 왜 필요한가?**
재무 리포트의 "연초부터 현재까지 누적", 성장 곡선, 진행률(누적 ÷ 전체) 같은 분석은 모두 누적합에서 출발합니다.

**3. 예시**

```sql
SELECT
    InvoiceId,
    InvoiceDate,
    Total,
    ROUND(SUM(Total) OVER (), 2) AS GrandTotal,           -- 전체 총합(고정)
    ROUND(SUM(Total) OVER (
        ORDER BY InvoiceDate
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ), 2) AS RunningTotal                                   -- 누적합(점점 커짐)
FROM   invoices
ORDER BY InvoiceDate;
```

**4. 헷갈리기 쉬운 점**
`ORDER BY`만 쓰고 `ROWS BETWEEN`을 생략해도 많은 DB에서 "처음부터 현재 행까지"가 기본 동작이지만, **명시하는 습관**을 들이면 의도가 분명해지고 DB 간 동작 차이로 인한 혼란을 줄일 수 있습니다.

**5. 한 줄 정리**
정렬 + "처음부터 현재까지" 범위 = 누적합.

> 비유: 적금 통장. 매달 입금액(각 행)은 그대로 보이고, 옆 칸에는 그달까지 모인 잔액(누적)이 찍힌다.

### 범위를 바꾸면 결과가 바뀐다

**1. 정의**
`ROWS BETWEEN`의 시작·끝 경계를 바꾸면 같은 함수라도 다른 "창"을 보게 됩니다.

**2. 왜 필요한가?**
"앞으로 쌓기" 대신 "현재부터 끝까지 남은 합"이 필요할 때가 있습니다. 경계 설정만 바꾸면 됩니다.

**3. 예시**

```sql
-- 현재 행부터 마지막 행까지 합 (뒤에서부터 빠지는 형태)
SUM(Total) OVER (
    ORDER BY InvoiceDate
    ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
)
```

**4. 헷갈리기 쉬운 점**
`UNBOUNDED PRECEDING ~ CURRENT ROW`(누적 증가)와 `CURRENT ROW ~ UNBOUNDED FOLLOWING`(잔여 감소)을 헷갈리면 그래프가 거꾸로 나옵니다. "어디서 시작해 어디서 끝나는가"를 항상 한 번 더 확인하세요.

**5. 한 줄 정리**
창의 시작과 끝을 바꾸면 누적의 방향이 바뀐다.

> 비유: 영화관에서 "내 앞줄까지 몇 명"과 "내 뒷줄까지 몇 명"은 같은 좌석을 봐도 전혀 다른 숫자다.

## 코드로 보기 — 월별 매출과 누적, 그리고 비중

윈도우 결과를 **다른 계산의 재료**로 쓰는 패턴입니다. 전체 합을 분모로 삼아 각 행의 비중을 구합니다.

```sql
SELECT
    BillingCountry,
    ROUND(SUM(Total), 2)                      AS CountryTotal,
    (SELECT SUM(Total) FROM invoices)         AS GrandTotal,     -- 전체 총합(스칼라 서브쿼리)
    ROUND(SUM(Total) / SUM(Total) OVER (), 4) AS RevenueRatio    -- 비중
FROM   invoices
GROUP BY BillingCountry
ORDER BY CountryTotal DESC;
```

**코드목적**
국가별 매출 합계를 구하고, 그 매출이 전체 매출에서 차지하는 비율을 함께 보여 주는 것이 목적입니다.

**해석**
`SUM(Total)`은 `GROUP BY`로 국가별 합을 만들고, `SUM(Total) OVER ()`는 그 집계 결과 전체의 합을 분모로 제공합니다. 나누면 국가별 매출 비중이 나옵니다. 비중이 높은 국가가 매출 집중도가 큰 핵심 시장입니다.

**실무 연결**
"상위 3개국이 전체 매출의 몇 %인가" 같은 **집중도 분석**, "각 제품이 카테고리 매출에서 차지하는 비중" 등 비율 리포트의 토대가 됩니다.

> ⚠️ 주의: `GROUP BY`와 `SUM() OVER()`를 한 쿼리에 섞을 때 분모가 의도와 다르게 잡혀 전체 합(GrandTotal)이 이상하게 나오는 경우가 있습니다. 그럴 때는 위처럼 **전체 합을 별도의 스칼라 서브쿼리**(`(SELECT SUM(Total) FROM invoices)`)로 분리해 분모로 쓰면 안전합니다.

## 직접 해보기

1. `SUM(Total) OVER ()`를 `AVG(Total) OVER ()`로 바꾸면 어떤 분석이 가능해질까요? 작성해 보세요.
2. 누적합 쿼리에서 `ROWS BETWEEN`을 `CURRENT ROW AND UNBOUNDED FOLLOWING`으로 바꿔 보고, 값이 어떻게 달라지는지 첫 행과 마지막 행을 비교해 보세요.
3. 국가별 매출 비중에 더해 **누적 비중**(상위부터 더한 비율)을 만들어, 어디까지가 전체의 80%인지 찾아보세요. (힌트: 정렬 후 누적합)

## 헷갈리기 쉬운 포인트

- **OVER() vs OVER(PARTITION BY)**: 빈 괄호는 전체 한 묶음, `PARTITION BY`는 그룹별 묶음. 분모를 "전체"로 할지 "그룹"으로 할지가 갈린다.
- **GROUP BY 집계 vs 윈도우 집계 혼용**: 한 쿼리에서 섞으면 분모(전체합)가 꼬일 수 있다. 애매하면 전체합은 스칼라 서브쿼리로 분리.
- **UNBOUNDED PRECEDING→CURRENT vs CURRENT→UNBOUNDED FOLLOWING**: 전자는 누적 증가, 후자는 잔여 감소.

## 연결되는 개념

- 이전 글: [순위 함수 3형제](02-ranking-functions.md) — 같은 `OVER`에 `ORDER BY`를 쓰는 또 다른 사례.
- 다음 글: [행 순서 함수 — LAG · LEAD](04-order-functions.md) — 누적이 아니라 "바로 이전/다음 행"을 끌어옵니다.
- 함께 보면 좋은 글: [비율과 등급](05-ratio-ntile-functions.md) — 비중 계산을 전용 함수로.
- 더 찾아볼 키워드: `frame clause`, `running total`, `RANGE vs ROWS`, `cumulative percentage`

## 셀프 체크

- [ ] `OVER()`로 전체 합/평균을 모든 행에 붙일 수 있다.
- [ ] 누적합을 `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`로 작성할 수 있다.
- [ ] 범위 경계를 바꾸면 누적 방향이 달라진다는 점을 안다.
- [ ] 윈도우 결과를 분모로 삼아 비율을 계산할 수 있다.
- [ ] GROUP BY와 OVER() 혼용 시 전체합이 꼬일 수 있어 서브쿼리로 분리하는 방법을 안다.

**복습 질문 및 답변**

- (기본) 누적합을 만들 때 `OVER` 안에 들어가는 범위 절은?
  → `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
- (이해확인) `SUM(x) OVER ()`와 `SUM(x) OVER (ORDER BY date ...)`의 차이는?
  → 전자는 모든 행에 같은 전체 합을 주고, 후자는 정렬 순서대로 점점 커지는 누적합을 준다.
- (응용) 각 국가 매출이 전체에서 차지하는 비중을 구할 때 분모가 이상하게 나온다면 어떻게 고칠까?
  → 전체 합을 `(SELECT SUM(...) FROM ...)` 스칼라 서브쿼리로 따로 구해 분모로 사용한다.

## 한 줄 정리

> 집계 함수에 `OVER`를 붙이면 행을 살린 채 합·평균을 붙일 수 있고, 범위(`ROWS BETWEEN`)를 조절하면 누적·잔여까지 자유자재로 만들 수 있다.
