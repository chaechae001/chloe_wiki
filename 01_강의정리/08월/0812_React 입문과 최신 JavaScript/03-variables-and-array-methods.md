# 변수 선언과 배열 메서드

React 코드는 데이터를 직접 변경하기보다 새 값으로 변환하는 일이 많아 `const`, `let`, `forEach`, `map`, `filter`의 역할 구분이 중요합니다.

## 핵심 키워드

`const` · `let` · `forEach` · `map` · `filter` · `immutability`

## 핵심 요약

- 기본적으로 `const`를 쓰고 재할당이 필요할 때 `let`을 사용합니다.
- `const`는 객체 내부 변경까지 막는 문법이 아닙니다.
- `forEach`는 반복 실행, `map`은 변환, `filter`는 선택에 사용합니다.
- React 목록 UI에서는 새 배열을 반환하는 메서드가 자주 쓰입니다.

## 1. const와 let

`const`는 같은 식별자에 다른 값을 재할당하지 못하게 합니다. `let`은 블록 범위에서 값을 다시 할당할 수 있습니다.

```javascript
const course = { title: "React" };
course.title = "React 기초"; // 객체 내부 변경은 가능

let page = 1;
page += 1;
```

변수가 가리키는 대상을 바꾸지 않는다면 `const`가 의도를 더 분명하게 합니다. 객체 상태를 불변하게 다루는 습관은 별도의 복사·변환 규칙으로 지켜야 합니다.

## 2. 배열 메서드의 목적

```javascript
const scores = [60, 80, 95];

scores.forEach((score) => console.log(score));
const doubled = scores.map((score) => score * 2);
const passed = scores.filter((score) => score >= 70);
```

| 메서드 | 목적 | 반환값 |
|---|---|---|
| `forEach` | 각 항목으로 부수 효과 수행 | `undefined` |
| `map` | 각 항목을 변환 | 같은 길이의 새 배열 |
| `filter` | 조건에 맞는 항목 선택 | 같거나 짧은 새 배열 |

## 대표 코드: 상품 데이터 가공

### 목적

판매 중인 상품만 골라 화면용 문자열로 변환합니다.

```javascript
const products = [
  { id: 1, name: "노트", active: true },
  { id: 2, name: "펜", active: false },
  { id: 3, name: "파일", active: true },
];

const labels = products
  .filter((product) => product.active)
  .map((product) => `${product.id}. ${product.name}`);

console.log(labels); // ["1. 노트", "3. 파일"]
```

### 코드 흐름과 결과

1. 원본 상품 배열을 준비합니다.
2. `filter`가 판매 중인 객체만 새 배열로 고릅니다.
3. `map`이 객체를 표시 문자열로 바꿉니다.
4. 원본 배열은 유지되어 다른 계산에도 사용할 수 있습니다.

### 실무 연결

API 응답을 카드 데이터로 바꾸거나 검색 조건에 맞는 항목만 렌더링할 때 같은 흐름을 사용합니다.

## 직접 해보기

1. `const` 객체의 프로퍼티 변경이 가능한 이유를 설명하세요.
2. 짝수만 골라 제곱한 새 배열을 만드세요.
3. 단순 출력에 `map`을 쓰는 것이 부적절한 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. `const`는 식별자의 재할당을 막지만 객체 자체를 자동으로 동결하지는 않습니다.
2. `numbers.filter((n) => n % 2 === 0).map((n) => n ** 2)`로 만들 수 있습니다.
3. map은 변환 결과 배열을 만들기 위한 메서드이므로 반환값을 버리는 출력 목적에는 forEach가 의도를 더 잘 나타냅니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| `const` vs 불변 객체 | const는 재할당만 막고 객체 내부의 불변성은 보장하지 않습니다. |
| `forEach` vs `map` | forEach는 반복 실행, map은 변환된 새 배열 생성이 목적입니다. |
| `map` vs `filter` | map은 모든 항목을 변환하고 filter는 일부 항목을 선택합니다. |

## 연결되는 개념

- 배열을 JSX로 바꾸는 흐름은 [명령형 DOM과 선언형 UI](02-imperative-vs-declarative-ui.md)에서 확인할 수 있습니다.
- 콜백 축약은 [화살표 함수와 구조 분해](04-arrow-functions-and-destructuring.md)에서 이어집니다.
- 복사 기반 갱신은 [전개 구문과 안전한 값 접근](05-spread-template-and-optional-chaining.md)에서 다룹니다.

## 셀프 체크

- [ ] const와 let의 재할당 차이를 안다.
- [ ] 세 배열 메서드의 반환값을 구분할 수 있다.
- [ ] 원본 배열을 유지하는 변환을 작성할 수 있다.

## 복습 질문 및 답변

### Q1. `forEach`의 반환값은 새 배열인가?

<details>
<summary>답</summary>

아닙니다. `forEach` 자체의 반환값은 `undefined`입니다.

</details>

### Q2. map 결과 배열의 길이는 원본과 같은가?

<details>
<summary>답</summary>

각 원소마다 하나의 결과를 반환하므로 일반적으로 같습니다.

</details>

### Q3. React 상태 배열을 직접 push하지 않는 이유는?

<details>
<summary>답</summary>

기존 참조를 직접 변경하면 변화 추적과 이전 상태 보존이 어려워지므로 새 배열을 만들어 갱신하는 방식이 안전합니다.

</details>

## 한 줄 정리

> const로 의도를 고정하고 배열 메서드로 원본을 보존하며 데이터를 선택·변환하는 습관이 React 코드의 기초입니다.
