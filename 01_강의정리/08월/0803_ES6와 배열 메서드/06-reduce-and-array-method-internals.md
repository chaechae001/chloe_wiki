# `reduce()`와 배열 메서드의 원리

> `reduce()`는 합계 전용 메서드가 아니라 이전 결과를 다음 요소와 결합하는 누적 규칙입니다.

`reduce` · `accumulator` · `initial value` · `custom map` · `custom filter`

## 핵심요약

- `reduce()`는 누적값과 현재 요소를 결합해 최종 결과 하나를 만듭니다.
- 초기값은 결과의 자료형과 빈 배열 처리 방식을 결정합니다.
- 합계뿐 아니라 객체, 배열, 그룹, 문자열도 누적할 수 있습니다.
- `map()`·`filter()`·`reduce()`는 반복문과 콜백 호출로 직접 구현할 수 있습니다.
- 직접 구현은 원리 학습에 유용하지만 실무에서는 검증된 내장 메서드를 우선합니다.

## 1. 누적값과 초기값

### 1) 정의

`reduce()`는 콜백이 반환한 결과를 다음 반복의 누적값으로 전달합니다.

```javascript
array.reduce((accumulator, current) => {
  return nextAccumulator;
}, initialValue);
```

### 2) 왜 필요한가

여러 요소를 합계, 최댓값, 객체, 그룹 같은 하나의 결과 구조로 조합할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const numbers = [10, 20, 30];

const total = numbers.reduce((sum, number) => {
  return sum + number;
}, 0);

console.log(total); // 60
```

초기값 0에서 시작해 10, 20, 30을 차례로 더합니다.

### 4) 쉬운 예시

장바구니 계산대에서 이전 상품까지의 합계에 현재 상품 금액을 계속 더하는 과정과 같습니다.

### 5) 코드 예시

```javascript
const cart = [
  { price: 3000, quantity: 2 },
  { price: 1500, quantity: 3 }
];

const total = cart.reduce(
  (sum, item) => sum + item.price * item.quantity,
  0
);

console.log(total); // 10500
```

### 6) 헷갈리는 점

초기값을 생략하면 첫 요소가 초기 누적값이 되고 반복은 두 번째 요소부터 시작합니다. 빈 배열에서는 오류가 날 수 있으므로 의도한 자료형의 초기값을 명시하는 습관이 안전합니다.

### 7) 한 줄 정리

> 초기값에서 출발해 현재 요소를 결합하고 새 누적값을 반환하는 것이 `reduce()`의 핵심입니다.

## 2. 합계보다 넓은 누적

### 1) 정의

누적값은 숫자일 필요가 없습니다. 배열이나 객체를 초기값으로 두면 분류와 그룹화 결과도 만들 수 있습니다.

### 2) 왜 필요한가

한 번의 순회로 데이터의 형태를 원하는 구조로 바꿀 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const words = ["apple", "ant", "banana"];

const grouped = words.reduce((groups, word) => {
  const key = word[0];
  groups[key] ??= [];
  groups[key].push(word);
  return groups;
}, {});
```

### 4) 쉬운 예시

서류를 첫 글자별 서랍에 분류하면서 서랍 안에 계속 추가하는 과정과 비슷합니다.

### 5) 코드 예시

```javascript
console.log(grouped);
// { a: ["apple", "ant"], b: ["banana"] }
```

### 6) 헷갈리는 점

콜백에서 누적값을 반환하지 않으면 다음 반복의 누적값이 `undefined`가 됩니다.

### 7) 한 줄 정리

> 초기값의 자료형이 `reduce()`가 만들 결과 구조의 출발점입니다.

## 코드로 보기 — 배열 메서드 직접 구현

```javascript
function customMap(array, transform) {
  const result = [];
  for (const item of array) {
    result.push(transform(item));
  }
  return result;
}

function customFilter(array, predicate) {
  const result = [];
  for (const item of array) {
    if (predicate(item)) result.push(item);
  }
  return result;
}

function customReduce(array, reducer, initialValue) {
  let accumulator = initialValue;
  for (const item of array) {
    accumulator = reducer(accumulator, item);
  }
  return accumulator;
}

const values = [1, 2, 3, 4];
console.log(customMap(values, (value) => value * 2));
console.log(customFilter(values, (value) => value % 2 === 0));
console.log(customReduce(values, (sum, value) => sum + value, 0));
```

### 코드 목적

세 배열 메서드가 반복, 콜백 호출, 결과 저장을 어떻게 조합하는지 확인합니다.

### 코드 흐름

1. `customMap()`은 변환값을 새 배열에 추가합니다.
2. `customFilter()`는 조건이 참일 때 원본값을 추가합니다.
3. `customReduce()`는 콜백 결과로 누적값을 갱신합니다.
4. 각 함수는 목적에 맞는 결과를 반환합니다.

### 실행 결과 해석

결과는 각각 `[2, 4, 6, 8]`, `[2, 4]`, `10`입니다. 순회 구조는 비슷하지만 저장하는 값과 최종 형태가 다릅니다.

### 실무 연결

내장 메서드의 동작을 이해하면 콜백 반환 누락, 잘못된 초기값, 의도와 다른 메서드 선택을 빠르게 디버깅할 수 있습니다. 실제 서비스 코드에서는 특별한 요구가 없다면 표준 내장 메서드를 사용하는 편이 명확하고 안전합니다.

## 직접 해보기

1. 숫자 합계의 초기값으로 무엇이 자연스러운가요?
2. 문자열 배열을 하나의 문장으로 누적하는 `reduce()`를 작성해 보세요.
3. 주문을 고객별 배열로 묶으려면 누적값의 초기 자료형을 무엇으로 두어야 할까요?

<details>
<summary>정답 보기</summary>

1. 덧셈의 항등원인 `0`이 자연스럽습니다.
2. 예: `words.reduce((text, word) => (text + " " + word).trim(), "")`처럼 작성할 수 있습니다.
3. 고객 ID를 키로 사용할 수 있도록 빈 객체 `{}`를 초기값으로 두는 방식이 적절합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 현재 요소 vs 누적값 | 원본 배열에서 읽은 값과 이전 반복이 만든 결과입니다. |
| 초기값 있음 vs 없음 | 반복 시작 위치와 빈 배열 처리, 누적 자료형이 달라집니다. |
| 직접 구현 vs 내장 메서드 | 원리 학습 목적과 실무의 검증된 표준 기능 사용 목적이 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [`forEach`·`map`·`filter`](05-foreach-map-and-filter.md)
- 다음에 이어지는 개념: 비동기 배열 처리와 함수 합성
- 함께 보면 좋은 키워드: `누적`, `그룹화`, `불변성`

## 셀프 체크

- [ ] 누적값과 현재 요소의 역할을 구분할 수 있다.
- [ ] 초기값이 필요한 이유를 설명할 수 있다.
- [ ] 숫자 외의 결과를 `reduce()`로 만들 수 있다.
- [ ] 세 배열 메서드의 반복 원리를 구현할 수 있다.
- [ ] 실무에서 내장 메서드를 우선하는 이유를 말할 수 있다.

### 복습 질문 및 답변

**Q1. `reduce()`는 합계를 구할 때만 사용하나요?**

<details>
<summary>답</summary>

아닙니다. 배열, 객체, 문자열, 그룹 등 콜백과 초기값으로 표현할 수 있는 다양한 결과를 만들 수 있습니다.

</details>

**Q2. 콜백에서 누적값을 반환하지 않으면 어떻게 되나요?**

<details>
<summary>답</summary>

다음 반복의 누적값이 `undefined`가 되어 이후 계산이 예상과 다르게 진행됩니다.

</details>

**Q3. 빈 배열에도 안전한 합계 코드를 만들려면 어떻게 해야 하나요?**

<details>
<summary>답</summary>

`numbers.reduce((sum, number) => sum + number, 0)`처럼 숫자 초기값을 명시합니다.

</details>

## 한 줄 정리

> `reduce()`는 초기 결과에 요소를 하나씩 결합하는 일반적인 누적 도구이며, 다른 배열 메서드의 내부 원리를 이해하는 열쇠입니다.
