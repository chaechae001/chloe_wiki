# `call`·`apply`·`bind`로 `this` 지정하기

> 하나의 함수를 여러 객체와 함께 쓰려면, 호출할 객체를 함수 밖에서 명시적으로 선택할 수 있어야 합니다.

`call` · `apply` · `bind` · `explicit binding` · `function reuse`

## 핵심요약

- `call()`과 `apply()`는 지정한 `this`로 함수를 즉시 실행합니다.
- `call()`은 추가 인자를 쉼표로, `apply()`는 배열 또는 배열과 유사한 값으로 전달합니다.
- `bind()`는 즉시 실행하지 않고 `this`가 고정된 새 함수를 반환합니다.
- 세 메서드는 객체와 독립적으로 작성한 함수를 여러 데이터에 재사용할 때 유용합니다.
- `bind()`로 만든 함수의 `this`는 일반적인 재호출 방식으로 바뀌지 않습니다.

## 1. `call()`과 `apply()`

### 1) 정의

두 메서드는 함수가 사용할 `this`를 첫 번째 인자로 받고 즉시 함수를 호출합니다. 차이는 나머지 인자를 전달하는 형태입니다.

### 2) 왜 필요한가

함수 로직을 특정 객체 안에 복제하지 않고도 현재 처리 대상만 바꾸어 실행할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
function format(prefix, suffix) {
  return `${prefix}${this.name}${suffix}`;
}

const item = { name: "키보드" };

console.log(format.call(item, "[", "]"));
console.log(format.apply(item, ["<", ">"]));
```

### 4) 쉬운 예시

같은 안내 방송 대본을 여러 매장의 정보와 결합해 즉시 읽되, 전달할 추가 정보가 낱개인지 묶음인지 선택하는 것과 같습니다.

### 5) 코드 예시

```javascript
function calculatePrice(quantity, discount) {
  return this.unitPrice * quantity - discount;
}

const product = { unitPrice: 12000 };

console.log(calculatePrice.call(product, 2, 1000));      // 23000
console.log(calculatePrice.apply(product, [3, 2000]));   // 34000
```

### 6) 헷갈리는 점

`apply()`의 두 번째 인자는 인자 목록 하나가 아니라 실제 함수 인자들로 펼쳐져 전달될 배열입니다.

### 7) 한 줄 정리

> 즉시 실행하면서 인자를 낱개로 주면 `call()`, 배열로 묶어 주면 `apply()`입니다.

## 2. `bind()`

### 1) 정의

`bind()`는 원본 함수를 호출하지 않고 지정한 `this`가 연결된 새 함수를 반환합니다.

### 2) 왜 필요한가

이벤트 등록, 타이머, 콜백 전달처럼 지금이 아니라 나중에 실행할 함수의 `this`를 미리 고정할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const logger = {
  prefix: "[INFO]",
};

function write(message) {
  return `${this.prefix} ${message}`;
}

const info = write.bind(logger);
console.log(info("저장 완료"));
```

### 4) 쉬운 예시

배송 기사와 주소를 미리 묶은 배송 요청서를 만들어 두고 원하는 시간에 실행하는 것과 같습니다.

### 5) 코드 예시

```javascript
function addTax(rate, price) {
  return price * (1 + rate) * this.currencyRate;
}

const market = { currencyRate: 1 };
const addLocalTax = addTax.bind(market, 0.1);

console.log(addLocalTax(10000)); // 11000
```

`bind()`는 `this`뿐 아니라 앞쪽 인자도 미리 고정할 수 있습니다.

### 6) 헷갈리는 점

`bind()`의 반환값은 함수입니다. 괄호로 다시 호출하지 않으면 본문은 실행되지 않습니다.

### 7) 한 줄 정리

> `bind()`는 실행할 대상과 일부 인자를 미리 묶은 새 함수를 만듭니다.

## 코드로 보기 — 공통 함수 재사용

```javascript
const basicPlan = { basePrice: 10000 };
const proPlan = { basePrice: 25000 };

function monthlyPrice(users, coupon = 0) {
  return this.basePrice * users - coupon;
}

const proForTeam = monthlyPrice.bind(proPlan, 4);

console.log(monthlyPrice.call(basicPlan, 2, 1000));
console.log(monthlyPrice.apply(proPlan, [2, 5000]));
console.log(proForTeam(10000));
```

### 코드 목적

하나의 가격 계산 함수를 서로 다른 요금제 객체와 즉시 또는 지연 방식으로 재사용합니다.

### 코드 흐름

1. `call()`로 기본 요금제와 개별 인자를 전달해 즉시 계산합니다.
2. `apply()`로 프로 요금제와 배열 인자를 전달해 즉시 계산합니다.
3. `bind()`로 프로 요금제와 사용자 수를 미리 고정합니다.
4. 나중에 쿠폰값만 전달해 새 함수를 실행합니다.

### 실행 결과 해석

각 결과는 같은 계산식을 사용하지만 `this.basePrice`와 전달 인자 조합에 따라 달라집니다.

### 실무 연결

공통 포매터, 권한 검사, 로깅 함수, 객체 메서드를 콜백으로 넘길 때 코드 복제를 줄이는 데 활용할 수 있습니다.

## 직접 해보기

1. `call()`과 `apply()`의 공통점은 무엇인가요?
2. `bind()`의 반환값을 변수에 저장만 하면 원본 함수가 실행되나요?
3. 나중에 실행할 콜백의 `this`를 특정 객체로 유지하려면 어떤 메서드가 적합한가요?

<details>
<summary>정답 보기</summary>

1. 첫 번째 인자로 `this`를 지정하고 함수를 즉시 실행합니다.
2. 실행되지 않습니다. 반환된 새 함수에 다시 `()`를 붙여 호출해야 합니다.
3. 실행을 미루면서 `this`를 고정하는 `bind()`가 적합합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `call()` vs `apply()` | 둘 다 즉시 실행하지만 추가 인자를 낱개 또는 배열로 전달합니다. |
| `call()` vs `bind()` | `call()`은 결과를, `bind()`는 나중에 실행할 새 함수를 반환합니다. |
| 바인딩 vs 객체 복사 | 바인딩은 함수 호출의 `this`를 정할 뿐 객체 데이터를 복사하지 않습니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [`this`와 호출 방식](03-this-binding-and-method-calls.md)
- 다음에 이어지는 개념: [콜백과 렉시컬 `this`](05-callbacks-arrow-functions-and-events.md)
- 함께 보면 좋은 키워드: `부분 적용`, `함수 재사용`, `콜백`

## 셀프 체크

- [ ] `call()`과 `apply()`의 인자 차이를 설명할 수 있다.
- [ ] 세 메서드의 실행 시점을 구분할 수 있다.
- [ ] `bind()`의 반환값이 함수임을 안다.
- [ ] 명시적 바인딩으로 함수를 재사용할 수 있다.
- [ ] 미리 고정된 인자가 어떻게 전달되는지 예측할 수 있다.

### 복습 질문 및 답변

**Q1. `apply()`는 왜 배열을 받나요?**

<details>
<summary>답</summary>

추가 인자들을 하나의 배열 형태로 받은 뒤 원본 함수의 각 매개변수에 순서대로 전달하기 위해서입니다.

</details>

**Q2. `bind()`가 원본 함수를 수정하나요?**

<details>
<summary>답</summary>

아닙니다. 원본은 그대로 두고 바인딩 정보가 적용된 새 함수를 반환합니다.

</details>

**Q3. `bind()`에 일부 인자를 미리 전달하면 어떤 장점이 있나요?**

<details>
<summary>답</summary>

반복되는 설정값을 고정하고 호출 시 달라지는 값만 받아 더 목적이 분명한 함수를 만들 수 있습니다.

</details>

## 한 줄 정리

> `call`·`apply`·`bind`는 함수와 호출 대상을 분리해 두고 실행 시점과 인자 형태에 맞게 다시 연결하는 도구입니다.
