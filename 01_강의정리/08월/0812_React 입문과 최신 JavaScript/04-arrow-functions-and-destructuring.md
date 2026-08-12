# 화살표 함수와 구조 분해

화살표 함수와 구조 분해는 React 코드에서 콜백과 컴포넌트 입력을 짧고 명확하게 표현하는 핵심 문법입니다.

## 핵심 키워드

`arrow function` · `implicit return` · `destructuring` · `props` · `default value`

## 핵심 요약

- 화살표 함수는 함수 표현식을 간결하게 작성합니다.
- 표현식 본문은 `return` 없이 값을 반환할 수 있습니다.
- 구조 분해는 객체나 배열에서 필요한 값을 이름으로 꺼냅니다.
- 기본값과 별칭을 사용하면 입력 처리 의도를 드러낼 수 있습니다.

## 1. 화살표 함수

```javascript
const add = (a, b) => a + b;
const makeUser = (name) => ({ name, active: true });
```

객체 리터럴을 암시적으로 반환할 때는 중괄호가 함수 본문으로 해석되지 않도록 소괄호로 감쌉니다. 화살표 함수는 자체 `this`를 만들지 않으므로 객체 메서드 대체로 무조건 사용하면 안 됩니다.

## 2. 구조 분해 할당

```javascript
const user = { id: 7, nickname: "새싹", role: "member" };
const { nickname, role: userRole, point = 0 } = user;

const colors = ["blue", "green"];
const [primary, secondary] = colors;
```

객체는 프로퍼티 이름을 기준으로, 배열은 위치를 기준으로 값을 꺼냅니다. `role: userRole`은 `role` 값을 새 변수 `userRole`에 저장합니다.

## 대표 코드: 컴포넌트 입력 읽기

### 목적

컴포넌트가 필요한 props만 구조 분해로 명확히 선언합니다.

```jsx
function ProductCard({ name, price, badge = "일반" }) {
  const formattedPrice = price.toLocaleString();

  return (
    <article>
      <strong>{name}</strong>
      <span>{formattedPrice}원</span>
      <em>{badge}</em>
    </article>
  );
}
```

### 코드 흐름과 결과

1. 함수 매개변수 위치에서 필요한 프로퍼티를 꺼냅니다.
2. 선택 입력 `badge`에는 기본값을 둡니다.
3. 가격을 표시용 문자열로 가공합니다.
4. JSX가 입력 데이터에 대응하는 카드 구조를 반환합니다.

### 실무 연결

이벤트 콜백, 배열 메서드, props와 함수 반환값을 간결하게 표현해 데이터 흐름을 읽기 쉽게 만듭니다.

## 직접 해보기

1. 객체를 암시적으로 반환할 때 소괄호가 필요한 이유를 설명하세요.
2. `{ title, count = 0 }`를 구조 분해하는 함수를 작성하세요.
3. 화살표 함수를 객체 메서드로 쓸 때 `this`를 주의해야 하는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 중괄호만 쓰면 함수 본문 블록으로 해석되므로 객체 리터럴임을 소괄호로 명확히 해야 합니다.
2. `const summarize = ({ title, count = 0 }) => `${title}: ${count}`;`처럼 작성할 수 있습니다.
3. 화살표 함수는 호출 객체가 아니라 선언된 주변 환경의 this를 사용하기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 명시적 return vs 암시적 return | 블록 본문은 return이 필요하고 표현식 본문은 결과를 자동 반환합니다. |
| 객체 구조 분해 vs 배열 구조 분해 | 객체는 이름, 배열은 위치를 기준으로 꺼냅니다. |
| 기본값 vs null 대체 | 구조 분해 기본값은 값이 `undefined`일 때 적용되고 `null`에는 적용되지 않습니다. |

## 연결되는 개념

- 콜백을 사용하는 메서드는 [변수 선언과 배열 메서드](03-variables-and-array-methods.md)에서 설명합니다.
- 객체 복사와 조합은 [전개 구문과 안전한 값 접근](05-spread-template-and-optional-chaining.md)에서 이어집니다.
- 컴포넌트 입력은 [컴포넌트와 State로 목록 설계](06-components-and-state.md)에서 활용합니다.

## 셀프 체크

- [ ] 화살표 함수의 두 본문 형식을 구분할 수 있다.
- [ ] 객체와 배열을 구조 분해할 수 있다.
- [ ] 기본값과 별칭을 사용할 수 있다.

## 복습 질문 및 답변

### Q1. 매개변수가 하나면 소괄호를 반드시 써야 하는가?

<details>
<summary>답</summary>

생략할 수 있지만 일관성과 타입 표기 확장을 위해 항상 쓰는 스타일도 흔합니다.

</details>

### Q2. 존재하지 않는 프로퍼티를 구조 분해하면?

<details>
<summary>답</summary>

기본값이 없으면 `undefined`, 기본값이 있으면 그 값이 사용됩니다.

</details>

### Q3. 함수 매개변수에서 구조 분해하는 장점은?

<details>
<summary>답</summary>

함수가 어떤 입력 프로퍼티를 사용하는지 선언부에서 바로 알 수 있습니다.

</details>

## 한 줄 정리

> 화살표 함수는 동작을 간결하게, 구조 분해는 데이터에서 필요한 입력을 명확하게 표현합니다.
