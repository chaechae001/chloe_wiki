# 내장 객체를 활용한 계산기 설계 패턴

> 계산식이 맞아도 입력 검증과 출력 포맷이 섞이면 재사용과 테스트가 어려워집니다.

`pure function` · `compound interest` · `return rate` · `break-even` · `formatting`

## 핵심요약

- 계산기는 입력 수집, 숫자 변환, 유효성 검사, 계산, 포매팅, 렌더링으로 나눕니다.
- 핵심 계산은 DOM을 모르는 순수 함수로 작성합니다.
- 비율은 퍼센트 입력과 소수 계산값을 구분합니다.
- 0으로 나누는 조건과 음수·빈 입력을 명시적으로 처리합니다.
- 표시 자릿수는 최종 출력에서만 적용해 중간 반올림을 피합니다.

## 1. 계산 함수와 UI 코드 분리

### 1) 정의

순수 계산 함수는 숫자를 받아 숫자를 반환하며 DOM을 직접 읽거나 바꾸지 않습니다. UI 코드는 폼 값을 수집하고 계산 함수를 호출한 뒤 결과를 표시합니다.

### 2) 왜 필요한가

계산식과 화면 코드가 섞이면 HTML이 없을 때 테스트하기 어렵고, 같은 공식을 다른 화면에서 재사용하기 힘듭니다.

### 3) 핵심 흐름 재구성

```text
폼 입력 → Number 변환 → 유효성 검사 → 계산 함수 → 표시 포맷 → DOM 출력
```

오류는 가능한 한 계산 전에 발견하고, 계산 함수는 의미 있는 실패 방식으로 처리합니다.

### 4) 쉬운 예시

식당에서 주문 접수, 재료 검수, 조리, 담기, 서빙을 나누는 것과 같습니다. 조리사가 주문서를 직접 화면에 그리기 시작하면 역할이 뒤섞입니다.

### 5) 코드 예시

```javascript
function calculateProfit({ buyPrice, currentPrice, quantity }) {
  const cost = buyPrice * quantity;
  const value = currentPrice * quantity;

  if (![cost, value].every(Number.isFinite) || cost <= 0) return null;

  const profit = value - cost;
  const returnRate = (profit / cost) * 100;
  return { cost, value, profit, returnRate };
}
```

### 6) 헷갈리는 점

수익률은 단순 가격 차이가 아니라 투입 금액 대비 손익 비율입니다. 수량은 수익금에는 영향을 주지만 동일 종목의 단순 수익률에는 상쇄될 수 있습니다.

### 7) 한 줄 정리

> 계산 함수는 숫자와 규칙만 책임지고, DOM과 문자열 출력은 바깥에서 담당합니다.

## 2. 복리와 손익분기점

### 1) 정의

복리 최종 금액은 원금 $P$, 연이율 $r$, 연간 복리 횟수 $n$, 기간 $t$에 대해 다음과 같습니다.

$$
A = P\left(1 + \frac{r}{n}\right)^{nt}
$$

손익분기점 매출액은 고정비를 공헌이익률로 나눈 값으로 표현할 수 있습니다.

$$
\text{BEP Sales} = \frac{\text{Fixed Cost}}{1 - \frac{\text{Variable Cost}}{\text{Revenue}}}
$$

### 2) 왜 필요한가

공식은 입력 단위와 분모 조건이 맞을 때만 의미가 있습니다. 퍼센트를 소수로 바꾸고, 복리 횟수는 양수인지, 매출이 0이 아닌지, 공헌이익률이 양수인지 확인해야 합니다.

### 3) 핵심 흐름 재구성

1. 모든 입력을 숫자로 변환합니다.
2. 금액·기간·빈도 범위를 검증합니다.
3. 퍼센트 이율을 100으로 나눕니다.
4. 원본 숫자로 계산합니다.
5. 금액과 비율을 출력 규칙에 맞게 포매팅합니다.

### 4) 쉬운 예시

복리는 이자가 원금 통에 다시 들어가 다음 이자 계산에 참여하는 구조입니다. 손익분기점은 판매 한 단위가 고정비 회수에 얼마나 기여하는지를 바탕으로 필요한 매출을 역산합니다.

### 5) 코드 예시

```javascript
function compoundAmount(principal, annualRatePercent, periodsPerYear, years) {
  const values = [principal, annualRatePercent, periodsPerYear, years];
  if (!values.every(Number.isFinite)) return null;
  if (principal < 0 || periodsPerYear <= 0 || years < 0) return null;

  const rate = annualRatePercent / 100;
  return principal * Math.pow(1 + rate / periodsPerYear, periodsPerYear * years);
}

function breakEvenSales(revenue, variableCost, fixedCost) {
  if (![revenue, variableCost, fixedCost].every(Number.isFinite)) return null;
  if (revenue <= 0 || fixedCost < 0) return null;

  const contributionMarginRatio = 1 - variableCost / revenue;
  if (contributionMarginRatio <= 0) return null;
  return fixedCost / contributionMarginRatio;
}
```

### 6) 헷갈리는 점

`Math.floor()`는 반올림이 아니라 내림입니다. 요구사항이 원 단위 버림인지 일반적인 반올림인지에 따라 `floor`와 `round`를 선택해야 합니다.

### 7) 한 줄 정리

> 금융·경영 공식은 계산식뿐 아니라 단위 변환과 계산 불가능 조건까지 함께 구현해야 합니다.

## 코드로 보기 — 폼 계산기를 작은 함수로 연결하기

```javascript
const form = document.querySelector("#compound-form");
const result = document.querySelector("#compound-result");

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const formData = new FormData(form);

  const amount = compoundAmount(
    Number(formData.get("principal")),
    Number(formData.get("rate")),
    Number(formData.get("frequency")),
    Number(formData.get("years"))
  );

  if (amount === null) {
    result.textContent = "입력값을 확인해 주세요.";
    return;
  }

  result.textContent = `${Math.round(amount).toLocaleString("ko-KR")}원`;
});
```

### 코드 목적

폼 입력을 수집하되 핵심 복리 계산은 독립 함수에 맡기고 결과만 화면에 표시합니다.

### 코드 흐름

1. 폼의 기본 제출을 막습니다.
2. `FormData`에서 이름별 입력값을 얻습니다.
3. 입력 문자열을 숫자로 변환해 계산 함수에 전달합니다.
4. 실패값과 정상값을 분기합니다.
5. 정상 금액을 반올림하고 지역 형식 문자열로 출력합니다.

### 실행 결과 해석

정상 입력이면 최종 금액이 천 단위 구분과 함께 표시됩니다. 계산 함수는 DOM 없이 별도 테스트할 수 있습니다.

### 실무 연결

대출 상환, 투자 시뮬레이션, 가격 마진, KPI 계산기, 견적 도구처럼 공식 기반 UI에 같은 구조를 적용할 수 있습니다.

## 직접 해보기

1. 매수가 10,000원, 현재가 12,000원, 수량 5주의 수익금과 수익률을 구해 보세요.
2. 연 6%, 월복리, 2년 계산에서 `r`, `n`, `t`에 들어갈 값을 적어 보세요.
3. 손익분기점 계산에서 변동비가 매출 이상이면 실패로 처리해야 하는 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. 원금은 50,000원, 현재가치는 60,000원, 수익금은 10,000원, 수익률은 20%입니다.
2. `r = 0.06`, `n = 12`, `t = 2`입니다.
3. 공헌이익률이 0 이하라 판매가 늘어도 고정비를 회수하는 정상적인 손익분기점이 나오지 않기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 수익금 vs 수익률 | 절대 금액 차이와 투입 금액 대비 비율입니다. |
| 퍼센트 입력 vs 계산 비율 | `6`과 `0.06`처럼 단위가 다릅니다. |
| `Math.floor()` vs `Math.round()` | 내림과 가장 가까운 정수 반올림입니다. |
| 계산값 vs 표시값 | 후속 계산용 숫자와 사람이 읽는 문자열입니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [String과 JSON](04-string-and-json.md)
- 다음에 이어지는 개념: 폼 검증과 상태 관리
- 함께 보면 좋은 키워드: `FormData`, `순수 함수`, `단위 테스트`

## 셀프 체크

- [ ] 계산기 처리 단계를 나눌 수 있다.
- [ ] DOM과 계산 로직을 분리할 수 있다.
- [ ] 퍼센트와 소수 비율을 변환할 수 있다.
- [ ] 분모가 0이 되는 조건을 검증할 수 있다.
- [ ] 표시 단계에서만 반올림과 포매팅을 적용할 수 있다.

### 복습 질문 및 답변

**Q1. 계산 함수를 순수하게 만들면 어떤 장점이 있나요?**

<details>
<summary>답</summary>

HTML 없이도 입력과 결과만으로 테스트할 수 있고 다른 화면에서도 재사용하기 쉽습니다.

</details>

**Q2. `FormData.get()`의 반환값을 왜 `Number()`로 바꾸나요?**

<details>
<summary>답</summary>

폼 값은 문자열이므로 덧셈이 문자열 연결로 동작하는 등의 타입 오류를 막기 위해서입니다.

</details>

**Q3. 합계 수익률을 종목별 수익률의 단순 평균으로 구하면 안 되는 이유는 무엇인가요?**

<details>
<summary>답</summary>

종목별 투자 금액이 다르므로 전체 매수액과 전체 현재가치를 기준으로 가중된 결과를 계산해야 하기 때문입니다.

</details>

## 한 줄 정리

> 신뢰할 수 있는 계산기는 공식보다 입력 단위, 실패 조건, 순수 계산, 최종 포매팅의 분리가 중요합니다.
