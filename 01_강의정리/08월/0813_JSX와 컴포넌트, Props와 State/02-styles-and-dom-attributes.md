# 스타일과 React DOM 속성

React의 DOM 속성은 JavaScript 객체의 프로퍼티처럼 작성하며, 스타일도 문자열이 아닌 객체로 전달합니다.

## 핵심 키워드

`className` · `style` · `camelCase` · `value` · `defaultValue`

## 핵심 요약

- CSS 클래스는 `class` 대신 `className`으로 지정합니다.
- 인라인 스타일은 camelCase 프로퍼티를 가진 객체로 전달합니다.
- JSX의 DOM 속성명은 브라우저 DOM 프로퍼티 규칙을 따릅니다.
- 현재 값을 React가 관리할지 초기값만 줄지 구분해야 합니다.

## 1. className과 style 객체

```jsx
function Notice({ important }) {
  const style = {
    backgroundColor: important ? "tomato" : "lightgray",
    padding: "12px",
  };

  return <p className="notice" style={style}>새 알림이 있습니다.</p>;
}
```

`background-color`는 JavaScript 식별자로 쓰기 어려워 `backgroundColor`로 작성합니다. 숫자 값 중 일부는 px 단위로 처리되지만 단위가 필요한 경우 문자열로 명시하면 의도가 분명합니다.

## 2. React DOM 속성 규칙

HTML과 이름이 다른 대표 속성은 다음과 같습니다.

| HTML | JSX | 이유 |
|---|---|---|
| `class` | `className` | JavaScript 문법과 DOM 프로퍼티 이름 사용 |
| `for` | `htmlFor` | JavaScript 예약어와 구분 |
| `onclick` | `onClick` | 이벤트 프로퍼티를 camelCase로 작성 |
| `tabindex` | `tabIndex` | DOM 프로퍼티 표기 사용 |

## 대표 코드: 제어 입력과 초기 입력

### 목적

React가 현재 값을 관리하는 입력과 브라우저가 초기값 이후 관리하는 입력을 비교합니다.

```jsx
import { useState } from "react";

function NameFields() {
  const [name, setName] = useState("");

  return (
    <form>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <input defaultValue="초기 별명" />
    </form>
  );
}
```

### 코드 흐름과 결과

1. 첫 입력은 State가 현재 값을 결정합니다.
2. 변경 이벤트가 State를 갱신해 화면과 데이터를 동기화합니다.
3. 둘째 입력은 처음에만 기본값을 받고 이후 값은 DOM이 관리합니다.
4. 같은 입력에서 제어 방식과 비제어 방식을 중간에 바꾸지 않는 것이 좋습니다.

### 실무 연결

폼 검증, 조건부 클래스, 접근성 속성, 사용자 입력 관리에 직접 연결됩니다.

## 직접 해보기

1. `class` 대신 `className`을 쓰는 이유를 설명하세요.
2. 글자색과 위쪽 여백을 가진 style 객체를 만드세요.
3. `value`만 있고 `onChange`가 없는 입력에서 생길 현상을 설명하세요.

<details>
<summary>정답 보기</summary>

1. JSX가 JavaScript와 DOM 프로퍼티 규칙을 사용하기 때문입니다.
2. `{ color: "navy", marginTop: "8px" }`처럼 작성합니다.
3. 값이 State에 고정되어 사용자가 입력해도 변경되지 않는 읽기 전용처럼 보일 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| className vs style | className은 CSS 규칙 재사용, style은 객체로 개별 인라인 값을 전달합니다. |
| value vs defaultValue | value는 현재값을 제어하고 defaultValue는 초기값만 제공합니다. |
| checked vs defaultChecked | 체크 상태를 계속 제어할지 초기 상태만 줄지의 차이입니다. |

## 연결되는 개념

- 기본 JSX 규칙은 [JSX 문법과 기본 규칙](01-jsx-syntax-and-rules.md)에서 확인할 수 있습니다.
- 입력 State는 [State와 불변 갱신](05-state-and-immutable-updates.md)에서 이어집니다.
- 목록 속성은 [목록 key와 입력 상태 설계](06-list-keys-and-form-state.md)에서 다룹니다.

## 셀프 체크

- [ ] className과 style을 올바르게 사용할 수 있다.
- [ ] JSX DOM 속성명을 구분할 수 있다.
- [ ] 현재값과 초기값 속성의 차이를 설명할 수 있다.

## 복습 질문 및 답변

### Q1. style 속성에 CSS 문자열을 넣는가?

<details>
<summary>답</summary>

아닙니다. JavaScript 스타일 객체를 전달합니다.

</details>

### Q2. 이벤트 속성은 어떤 표기로 작성하는가?

<details>
<summary>답</summary>

`onClick`, `onChange`처럼 camelCase로 작성하고 함수 값을 전달합니다.

</details>

### Q3. 인라인 스타일만 사용하면 생길 수 있는 단점은?

<details>
<summary>답</summary>

스타일 재사용과 미디어 쿼리·가상 선택자 활용이 어렵고 컴포넌트 코드가 복잡해질 수 있습니다.

</details>

## 한 줄 정리

> React DOM 속성은 JavaScript 규칙으로 작성하며 현재값과 초기값의 관리 주체를 명확히 구분해야 합니다.
