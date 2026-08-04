# `forEach()`·`map()`·`filter()`

> 세 메서드의 차이는 반복 횟수가 아니라 “무엇을 결과로 남겨야 하는가”에 있습니다.

`forEach` · `map` · `filter` · `callback return` · `new array`

## 핵심요약

- `forEach()`는 각 요소에 작업을 실행하며 메서드 반환값은 `undefined`입니다.
- `map()`은 모든 요소를 변환해 원본과 같은 길이의 새 배열을 만듭니다.
- `filter()`는 조건이 참인 요소만 선택해 새 배열을 만듭니다.
- 콜백의 `return`은 메서드별로 변환값 또는 선택 조건이 됩니다.
- 원본 배열은 기본적으로 유지되지만 콜백 안에서 객체를 직접 변경할 수는 있습니다.

## 1. 결과 형태로 메서드 고르기

### 1) 정의

세 메서드는 모두 배열을 순회하지만 목적과 반환 결과가 다릅니다.

| 원하는 작업 | 메서드 | 결과 |
|---|---|---|
| 각 요소마다 출력·등록 같은 작업 실행 | `forEach()` | `undefined` |
| 모든 요소를 다른 값으로 변환 | `map()` | 같은 길이의 새 배열 |
| 조건에 맞는 요소만 선택 | `filter()` | 길이가 같거나 짧은 새 배열 |

### 2) 왜 필요한가

의도를 메서드 이름에 드러내면 반복문의 초기화·인덱스·`push()`보다 핵심 규칙에 집중할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const numbers = [1, 2, 3, 4];

numbers.forEach((number) => console.log(number));
const doubled = numbers.map((number) => number * 2);
const evens = numbers.filter((number) => number % 2 === 0);
```

### 4) 쉬운 예시

학생 명단을 한 명씩 호명하면 `forEach()`, 이름을 명찰 문구로 바꾸면 `map()`, 출석한 학생만 남기면 `filter()`에 해당합니다.

### 5) 코드 예시

```javascript
const products = [
  { name: "pen", price: 1000 },
  { name: "note", price: 3000 },
  { name: "bag", price: 20000 }
];

const affordableNames = products
  .filter((product) => product.price <= 5000)
  .map((product) => product.name);

console.log(affordableNames); // ["pen", "note"]
```

### 6) 헷갈리는 점

메서드 체인의 순서도 의미가 있습니다. 먼저 `filter()`하면 이후 `map()`이 처리할 요소 수가 줄어들 수 있습니다.

### 7) 한 줄 정리

> 반복·변환·선택 중 필요한 결과를 먼저 정하면 메서드가 자연스럽게 결정됩니다.

## 2. 콜백의 반환값

### 1) 정의

`map()`은 콜백 반환값을 새 요소로 사용하고, `filter()`는 반환값을 참·거짓으로 평가합니다.

### 2) 왜 필요한가

중괄호를 쓴 화살표 함수에서 `return`을 빼먹으면 `map()`에는 `undefined`가 쌓이고 `filter()`에서는 모든 요소가 탈락할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const wrong = [1, 2, 3].map((number) => {
  number * 2;
});

console.log(wrong); // [undefined, undefined, undefined]
```

### 4) 쉬운 예시

검사 담당자가 결과표에 판정값을 적지 않으면 다음 단계는 무엇을 남길지 결정할 수 없습니다. 콜백의 `return`이 그 판정표입니다.

### 5) 코드 예시

```javascript
const correct = [1, 2, 3].map((number) => {
  return number * 2;
});
```

### 6) 헷갈리는 점

`forEach()` 콜백에서 값을 반환해도 메서드의 결과 배열이 만들어지지 않습니다. 변환 결과가 필요하면 `map()`을 사용해야 합니다.

### 7) 한 줄 정리

> 배열 메서드에서 콜백의 반환값은 다음 데이터가 만들어지는 규칙입니다.

## 코드로 보기 — 객체 배열을 HTML 목록으로 변환

```javascript
const people = [
  { name: "Mina", age: 28 },
  { name: "Jin", age: 15 },
  { name: "Ara", age: 31 }
];

const adultList = people
  .filter((person) => person.age >= 18)
  .map((person) => `<li>${person.name}</li>`)
  .join("");

console.log(adultList);
// <li>Mina</li><li>Ara</li>
```

### 코드 목적

객체 배열에서 조건에 맞는 사람을 선택하고 화면에 넣을 문자열로 변환합니다.

### 코드 흐름

1. `filter()`가 성인 객체만 남깁니다.
2. `map()`이 각 객체를 `<li>` 문자열로 바꿉니다.
3. `join("")`이 문자열 배열을 하나로 연결합니다.
4. 완성된 문자열을 출력합니다.

### 실행 결과 해석

원본에는 세 사람이 있지만 결과 문자열에는 조건을 통과한 두 사람만 포함됩니다.

### 실무 연결

API 목록을 검색 조건으로 거른 뒤 카드·표·목록용 화면 데이터로 변환하는 전형적인 흐름입니다. 실제 DOM 출력에서는 안전한 노드 생성과 데이터 이스케이프도 고려해야 합니다.

## 직접 해보기

1. 각 요소를 콘솔에만 출력하려면 어떤 메서드가 적절한가요?
2. 숫자 배열을 문자열 배열로 바꾸려면 어떤 메서드를 사용하나요?
3. 활성 사용자만 선택해 이름 목록을 만드는 메서드 순서를 작성해 보세요.

<details>
<summary>정답 보기</summary>

1. 반환 결과가 필요 없는 반복 작업이므로 `forEach()`가 적절합니다.
2. 모든 요소를 변환해 새 배열을 만들어야 하므로 `map()`을 사용합니다.
3. `filter()`로 활성 사용자를 남긴 뒤 `map()`으로 이름을 꺼냅니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `forEach()` vs `map()` | 작업만 실행하는지 변환된 새 배열이 필요한지 다릅니다. |
| `map()` vs `filter()` | 요소 값을 바꾸는지 요소를 남길지 결정하는지 다릅니다. |
| 메서드 반환값 vs 콜백 반환값 | 메서드 전체 결과와 각 요소 처리 함수의 결과는 역할이 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [고차 함수와 콜백](04-higher-order-functions-and-callbacks.md)
- 다음에 이어지는 개념: [`reduce`와 배열 메서드의 원리](06-reduce-and-array-method-internals.md)
- 함께 보면 좋은 키워드: `메서드 체이닝`, `불변성`, `join`

## 셀프 체크

- [ ] 세 메서드를 결과 형태로 구분할 수 있다.
- [ ] `map()` 결과 길이를 예측할 수 있다.
- [ ] `filter()` 콜백의 반환 의미를 설명할 수 있다.
- [ ] 화살표 함수 블록에서 `return`을 작성할 수 있다.
- [ ] 객체 배열을 선택하고 변환하는 체인을 작성할 수 있다.

### 복습 질문 및 답변

**Q1. `forEach()`를 변수에 저장하면 배열이 들어가나요?**

<details>
<summary>답</summary>

아닙니다. `forEach()` 자체의 반환값은 `undefined`입니다.

</details>

**Q2. `map()`은 원본 배열의 길이와 같은 결과를 만드나요?**

<details>
<summary>답</summary>

네. 각 원본 요소마다 하나의 콜백 반환값을 새 배열에 넣으므로 길이가 같습니다.

</details>

**Q3. `filter()` 다음 `map()`을 연결하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

먼저 필요한 요소만 선택하고, 선택된 요소를 원하는 출력 형태로 변환하기 위해서입니다.

</details>

## 한 줄 정리

> `forEach()`는 실행, `map()`은 변환, `filter()`는 선택이라는 결과 중심의 기준으로 구분합니다.
