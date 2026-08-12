# 컴포넌트와 State로 목록 설계

목록 UI를 React로 옮길 때 핵심 변화는 DOM 문자열을 직접 추가하는 대신 배열 상태를 바꾸고 컴포넌트가 화면을 계산하게 하는 것입니다.

## 핵심 키워드

`component` · `state` · `event` · `list rendering` · `key`

## 핵심 요약

- State는 컴포넌트가 기억하고 변경에 따라 화면을 다시 그릴 값입니다.
- 이벤트 처리기는 상태 변경을 요청하고 JSX는 상태를 화면으로 변환합니다.
- 목록은 배열의 `map`으로 렌더링합니다.
- 항목의 key는 배열 위치보다 안정적인 식별자를 사용하는 것이 좋습니다.

## 1. 문자열 조립에서 데이터 중심으로

직접 DOM 방식은 HTML 문자열 생성과 삽입 시점을 관리합니다. React에서는 할 일 배열을 상태로 두고 각 항목을 컴포넌트로 표현합니다.

```jsx
function TodoItem({ todo }) {
  return <li>{todo.title}</li>;
}

function TodoList({ todos }) {
  return <ul>{todos.map((todo) => <TodoItem key={todo.id} todo={todo} />)}</ul>;
}
```

컴포넌트는 재사용 가능한 UI 블록이고 State는 시간이 지나며 바뀌는 데이터입니다.

## 2. 상태를 직접 변경하지 않기

새 항목을 추가할 때 기존 배열에 `push`하고 같은 배열을 재사용하기보다 새 배열을 만듭니다.

```jsx
setTodos((previous) => [
  ...previous,
  { id: crypto.randomUUID(), title: input, done: false },
]);
```

이전 상태에 의존하는 갱신은 함수형 업데이트를 사용하면 연속된 변경에서도 최신 상태를 기준으로 계산할 수 있습니다.

## 대표 코드: 작은 Todo 컴포넌트

### 목적

입력 이벤트로 새 항목을 상태에 추가하고 목록을 선언적으로 렌더링합니다.

```jsx
import { useState } from "react";

function TodoApp() {
  const [title, setTitle] = useState("");
  const [todos, setTodos] = useState([]);

  const addTodo = () => {
    const trimmed = title.trim();
    if (!trimmed) return;

    setTodos((previous) => [
      ...previous,
      { id: crypto.randomUUID(), title: trimmed },
    ]);
    setTitle("");
  };

  return (
    <main>
      <input value={title} onChange={(event) => setTitle(event.target.value)} />
      <button onClick={addTodo}>추가</button>
      <ul>{todos.map((todo) => <li key={todo.id}>{todo.title}</li>)}</ul>
    </main>
  );
}
```

### 코드 흐름과 결과

1. 입력값과 목록을 각각 State로 선언합니다.
2. 입력 이벤트가 title 상태를 갱신합니다.
3. 버튼 이벤트가 검증된 새 객체를 새 배열에 추가합니다.
4. 배열이 바뀌면 목록 JSX가 새 데이터로 다시 계산됩니다.

### 실무 연결

폼, 검색 조건, 선택 목록 등 사용자의 상호작용으로 UI가 달라지는 기능의 기본 구조입니다.

## 직접 해보기

1. State와 일반 지역 변수의 차이를 설명하세요.
2. 특정 id를 제외해 항목을 삭제하는 코드를 작성하세요.
3. 배열 인덱스를 key로 쓸 때 생길 수 있는 문제를 설명하세요.

<details>
<summary>정답 보기</summary>

1. State는 렌더링 사이에 값이 보존되고 setter 호출이 화면 갱신을 예약하지만 지역 변수는 렌더링마다 다시 만들어집니다.
2. `setTodos((previous) => previous.filter((todo) => todo.id !== targetId));`로 작성할 수 있습니다.
3. 순서가 바뀌거나 항목이 삭제되면 기존 UI 상태가 다른 항목에 잘못 연결될 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| Props vs State | Props는 부모가 전달한 입력, State는 컴포넌트가 기억하고 갱신하는 값입니다. |
| 이벤트 핸들러 전달 vs 호출 | `onClick={addTodo}`는 함수를 전달하고 `onClick={addTodo()}`는 렌더링 중 즉시 호출합니다. |
| 배열 변경 vs 새 배열 생성 | 직접 변경은 같은 참조를 유지하고 새 배열 생성은 변경을 명확히 드러냅니다. |

## 연결되는 개념

- 선언형 목록의 원리는 [명령형 DOM과 선언형 UI](02-imperative-vs-declarative-ui.md)에서 설명합니다.
- 배열 갱신은 [변수 선언과 배열 메서드](03-variables-and-array-methods.md)에서 확인할 수 있습니다.
- 실행 환경은 [React 프로젝트 환경과 구조](07-react-project-setup.md)에서 이어집니다.

## 셀프 체크

- [ ] Props와 State를 구분할 수 있다.
- [ ] 새 배열로 상태를 갱신할 수 있다.
- [ ] 목록 key의 역할을 설명할 수 있다.

## 복습 질문 및 답변

### Q1. State setter 호출 직후 변수를 직접 읽으면 항상 새 값인가?

<details>
<summary>답</summary>

아닙니다. 상태 갱신은 다음 렌더링을 예약하며 현재 실행 중인 렌더링의 값은 스냅샷처럼 유지됩니다.

</details>

### Q2. 목록 렌더링에 주로 사용하는 배열 메서드는?

<details>
<summary>답</summary>

각 데이터 항목을 JSX 요소로 변환하는 `map`입니다.

</details>

### Q3. 이전 상태에 의존하는 갱신에서 함수형 setter가 유리한 이유는?

<details>
<summary>답</summary>

React가 제공하는 최신 이전 상태를 인수로 받아 연속된 갱신을 안전하게 계산할 수 있기 때문입니다.

</details>

## 한 줄 정리

> React 목록은 배열 State를 단일 기준으로 두고 이벤트가 새 상태를 만들며 JSX가 화면을 계산하는 구조입니다.
