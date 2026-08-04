# 고차 함수와 콜백

> 함수를 값처럼 전달할 수 있으면 반복 구조와 “무엇을 할지”를 분리해 재사용할 수 있습니다.

`higher-order function` · `callback` · `function value` · `event handler` · `closure`

## 핵심요약

- JavaScript 함수는 변수에 저장하고 인자로 전달하며 반환할 수 있습니다.
- 고차 함수는 함수를 받거나 반환하는 함수입니다.
- 콜백은 다른 함수에 전달되어 그 함수가 정한 시점에 실행됩니다.
- 배열 메서드는 반복을 담당하고 콜백은 요소별 규칙을 담당합니다.
- 콜백을 호출한 결과와 콜백 함수 자체를 혼동하지 않아야 합니다.

## 1. 함수를 값으로 다루기

### 1) 정의

JavaScript에서 함수는 숫자나 문자열처럼 변수에 담고 다른 함수에 전달할 수 있는 값입니다.

### 2) 왜 필요한가

반복, 지연 실행, 이벤트 처리 같은 공통 흐름과 실제 동작을 분리하면 같은 구조에 여러 규칙을 끼워 넣을 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
function repeat(count, action) {
  for (let index = 0; index < count; index += 1) {
    action(index);
  }
}

repeat(3, (index) => console.log(`작업 ${index + 1}`));
```

`repeat()`는 반복 횟수와 순서를 담당하고, 전달된 콜백은 매번 무엇을 할지 결정합니다.

### 4) 쉬운 예시

커피 머신은 물을 데우고 추출하는 공통 순서를 담당하고, 선택한 캡슐이 실제 맛을 결정합니다. 공통 순서가 고차 함수, 끼워 넣은 동작이 콜백과 비슷합니다.

### 5) 코드 예시

```javascript
const print = (value) => console.log(value);
repeat(2, print);
```

`print`를 전달해야 하며 `print()`를 쓰면 함수를 먼저 실행한 결과를 전달하게 됩니다.

### 6) 헷갈리는 점

콜백은 항상 비동기인 것이 아닙니다. `map()`과 `filter()`의 콜백은 배열을 순회하는 동안 동기적으로 실행됩니다.

### 7) 한 줄 정리

> 고차 함수는 실행 구조를, 콜백은 그 구조 안의 구체적인 동작을 담당합니다.

## 2. 함수를 반환하는 함수

### 1) 정의

고차 함수는 함수를 반환해 일부 설정이 기억된 새 함수를 만들 수도 있습니다.

### 2) 왜 필요한가

비슷한 규칙을 매번 다시 작성하지 않고 설정만 바꾼 전용 함수를 만들 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
function createMultiplier(factor) {
  return (value) => value * factor;
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

반환된 함수는 `createMultiplier()` 실행이 끝난 뒤에도 `factor`를 기억합니다.

### 4) 쉬운 예시

도장을 한 번 제작하면 이후에는 같은 배율이나 형식을 반복 적용할 수 있는 것과 비슷합니다.

### 5) 코드 예시

```javascript
const isAtLeast = (minimum) => (value) => value >= minimum;
const isAdult = isAtLeast(18);

console.log(isAdult(20)); // true
```

### 6) 헷갈리는 점

반환된 함수가 바깥 함수의 변수를 기억하는 현상은 렉시컬 스코프와 클로저에 기반합니다.

### 7) 한 줄 정리

> 함수를 반환하면 설정을 기억하는 재사용 가능한 규칙을 만들 수 있습니다.

## 코드로 보기 — 조건을 전달하는 목록 처리

```javascript
function select(items, predicate) {
  const selected = [];

  for (const item of items) {
    if (predicate(item)) {
      selected.push(item);
    }
  }

  return selected;
}

const scores = [55, 80, 67, 95];
const passed = select(scores, (score) => score >= 70);

console.log(passed); // [80, 95]
```

### 코드 목적

반복 로직과 선택 조건을 분리해 `filter()`가 동작하는 기본 원리를 이해합니다.

### 코드 흐름

1. 결과 배열을 준비합니다.
2. 모든 요소를 순회합니다.
3. 콜백 결과가 참인 요소만 추가합니다.
4. 선택된 새 배열을 반환합니다.

### 실행 결과 해석

조건 함수는 숫자가 70 이상인지 판단하고, 고차 함수는 통과한 값만 모읍니다.

### 실무 연결

권한 검사, 폼 검증, 데이터 필터 규칙, 이벤트 핸들러 등록처럼 동작을 외부에서 주입하는 설계에 쓰입니다.

## 직접 해보기

1. 고차 함수의 두 가지 조건을 적어 보세요.
2. `repeat(3, console.log)`와 `repeat(3, console.log())`의 차이를 설명해 보세요.
3. 최소 길이를 받아 문자열 검사 함수를 반환하는 함수를 설계해 보세요.

<details>
<summary>정답 보기</summary>

1. 함수를 인자로 받거나 함수를 반환하면 고차 함수입니다.
2. 첫 코드는 함수 자체를 전달하고, 두 번째 코드는 즉시 호출한 반환값을 전달합니다.
3. 예: `const hasMinLength = (min) => (text) => text.length >= min;`처럼 작성할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 함수 참조 vs 함수 호출 | `fn`은 함수 값이고 `fn()`은 함수를 실행한 결과입니다. |
| 고차 함수 vs 콜백 | 함수를 받거나 반환하는 쪽과 전달되어 실행되는 쪽입니다. |
| 동기 콜백 vs 비동기 콜백 | 현재 흐름에서 바로 실행되는지 나중 이벤트나 작업 완료 후 실행되는지 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [구조 분해와 Rest·Spread](03-destructuring-rest-and-spread.md)
- 다음에 이어지는 개념: [`forEach`·`map`·`filter`](05-foreach-map-and-filter.md)
- 함께 보면 좋은 키워드: `일급 객체`, `클로저`, `함수 합성`

## 셀프 체크

- [ ] 함수를 값으로 전달할 수 있다.
- [ ] 고차 함수와 콜백의 역할을 구분할 수 있다.
- [ ] 함수 참조와 호출 결과를 구분할 수 있다.
- [ ] 콜백이 항상 비동기는 아니라는 점을 설명할 수 있다.
- [ ] 함수를 반환하는 함수를 작성할 수 있다.

### 복습 질문 및 답변

**Q1. `map()`은 왜 고차 함수인가요?**

<details>
<summary>답</summary>

각 요소를 어떻게 변환할지 결정하는 콜백 함수를 인자로 받기 때문입니다.

</details>

**Q2. 콜백은 누가 호출하나요?**

<details>
<summary>답</summary>

콜백을 전달받은 고차 함수나 이벤트 시스템이 정해진 시점과 인자로 호출합니다.

</details>

**Q3. 반복문과 조건을 한 함수에 모두 고정하는 것보다 콜백을 받으면 무엇이 좋아지나요?**

<details>
<summary>답</summary>

반복 구조를 재사용하면서 선택·변환 규칙만 바꿀 수 있어 중복 코드가 줄고 테스트하기 쉬워집니다.

</details>

## 한 줄 정리

> 함수를 데이터처럼 전달하면 공통 실행 흐름과 바뀌는 규칙을 분리할 수 있습니다.
