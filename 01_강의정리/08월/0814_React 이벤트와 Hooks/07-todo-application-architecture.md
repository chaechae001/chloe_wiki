# 통합 Todo 애플리케이션 설계

작은 Todo 앱은 React의 핵심을 한 번에 연습하기 좋은 모델입니다. 입력, 이벤트, State, Props, 목록 렌더링, 불변 갱신, 파생값, 스타일 상태를 하나의 데이터 흐름으로 연결할 수 있습니다.

`todo list` · `immutable update` · `list key` · `derived state` · `component responsibility`

## 핵심요약

- 목록 State는 공통 부모가 소유하고 입력 폼과 목록 뷰에 필요한 값과 콜백을 전달합니다.
- 항목 추가·완료·삭제는 기존 배열이나 객체를 직접 수정하지 않고 새 값을 반환합니다.
- 목록 `key`는 렌더링 사이에 항목의 정체성을 안정적으로 나타내야 합니다.
- 입력값 검증과 제출 로직은 폼 컴포넌트에, 목록 변경 규칙은 State 소유자에 둡니다.
- 개수 제한 같은 파생값은 원본 State에서 계산하며 별도 State로 중복 저장하지 않습니다.

## 1. 컴포넌트 책임 나누기

간단한 구조는 다음과 같습니다.

- `TodoApp`: 목록 State와 추가·완료·삭제 규칙을 소유합니다.
- `TodoForm`: 입력 State를 관리하고 유효한 값을 부모에 전달합니다.
- `TodoList`: 항목을 렌더링하고 사용자 행동을 콜백으로 알립니다.

데이터는 부모에서 자식으로 내려가고, 이벤트는 콜백 호출로 올라옵니다. 이 단방향 흐름은 변경 원인을 추적하기 쉽게 만듭니다.

## 2. 불변하게 목록 갱신하기

```jsx
const addTodo = (text) => {
  setTodos((current) => [
    ...current,
    { id: crypto.randomUUID(), text, completed: false },
  ]);
};

const completeTodo = (id) => {
  setTodos((current) =>
    current.map((todo) =>
      todo.id === id ? { ...todo, completed: true } : todo
    )
  );
};

const removeTodo = (id) => {
  setTodos((current) => current.filter((todo) => todo.id !== id));
};
```

`map`은 변경된 항목만 새 객체로 만들고, `filter`는 제거할 항목을 제외한 새 배열을 반환합니다. 인덱스 대신 안정적인 `id`를 사용하면 항목 순서가 달라져도 올바른 대상을 찾을 수 있습니다.

## 3. 파생값과 UI 상태

완료 개수나 입력 제한 여부는 기존 목록에서 계산할 수 있습니다.

```jsx
const completedCount = todos.filter((todo) => todo.completed).length;
const isLimitReached = todos.length >= 8;
```

이 값을 별도 State에 저장하면 원본 목록과 서로 어긋날 수 있습니다. 계산 비용이 실제로 크지 않다면 렌더링 중 바로 계산하는 것이 단순합니다. 비용이 측정될 정도로 크다면 `useMemo`를 검토할 수 있습니다.

완료 상태는 클래스 이름이나 속성으로 CSS에 전달합니다.

```jsx
<li className={todo.completed ? "completed" : ""}>{todo.text}</li>
```

데이터가 스타일을 결정하게 하면 DOM을 직접 찾아 클래스를 바꾸는 코드가 필요 없습니다.

## 코드로 보기 — 핵심 목록 흐름

```jsx
import { useState } from "react";

function TodoApp() {
  const [todos, setTodos] = useState([]);

  const addTodo = (text) => {
    setTodos((current) => [
      ...current,
      { id: crypto.randomUUID(), text, completed: false },
    ]);
  };

  const toggleTodo = (id) => {
    setTodos((current) =>
      current.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const removeTodo = (id) => {
    setTodos((current) => current.filter((todo) => todo.id !== id));
  };

  return (
    <main>
      <TodoForm onAdd={addTodo} disabled={todos.length >= 8} />
      <TodoList todos={todos} onToggle={toggleTodo} onRemove={removeTodo} />
    </main>
  );
}
```

### 코드 목적

목록 데이터와 변경 규칙을 부모에 모으고 하위 컴포넌트에는 필요한 인터페이스만 전달합니다.

### 코드 흐름

1. 폼이 검증된 문자열을 `onAdd`로 보냅니다.
2. 부모가 고유 id를 가진 새 항목을 목록에 추가합니다.
3. 목록 뷰가 id와 함께 완료 또는 삭제 콜백을 호출합니다.
4. 부모가 `map`이나 `filter`로 새 배열을 만들어 State를 갱신합니다.

### 실행 결과 해석

State가 바뀔 때 React가 새 목록을 렌더링합니다. 항목의 `completed` 값은 텍스트와 스타일을 함께 결정하고, 제한에 도달하면 폼을 비활성화할 수 있습니다.

### 실무 연결

장바구니, 알림 목록, 관리자 테이블, 작업 보드처럼 항목을 추가·수정·삭제하는 CRUD UI의 축소판입니다.

## 직접 해보기

1. 완료 항목만 보여 주는 필터를 추가해 보세요.
2. 모든 항목을 미완료로 되돌리는 함수를 불변 갱신으로 작성해 보세요.
3. 목록 제한에 도달하면 입력을 비활성화하고 안내 문구를 표시해 보세요.

<details>
<summary>정답 보기</summary>

1. `todos.filter(todo => todo.completed)`로 파생 목록을 만들고 선택한 필터에 따라 렌더링합니다.
2. `setTodos(current => current.map(todo => ({ ...todo, completed: false })))`를 사용합니다.
3. `const isLimitReached = todos.length >= limit`를 계산해 폼의 `disabled`와 조건부 안내 문구에 같은 값을 전달합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 원본 State vs 파생값 | 원본 데이터는 State로 저장하고, 그 데이터에서 계산 가능한 값은 필요할 때 계산합니다. |
| 배열 인덱스 vs 안정적인 id | 인덱스는 순서가 바뀌면 대상을 가리키는 의미가 달라질 수 있지만 id는 항목 정체성을 유지합니다. |
| 직접 변경 vs 불변 갱신 | 직접 변경은 기존 참조를 오염시키고, 불변 갱신은 새 배열·객체로 변경 경계를 드러냅니다. |

## 연결되는 개념

- 폼 구성은 [폼과 제어 입력](02-forms-and-controlled-inputs.md)에서 확인하세요.
- 부모·자식 책임 분리는 [콜백 Props와 컴포넌트 이벤트](03-callback-props-and-component-events.md)에서 설명합니다.
- 파생값 최적화는 [메모이제이션과 ref](05-memoization-and-refs.md)에서 이어집니다.

## 셀프 체크

- [ ] 목록 State의 적절한 소유 위치를 정할 수 있다.
- [ ] 추가·수정·삭제를 불변 갱신으로 구현할 수 있다.
- [ ] 안정적인 key가 필요한 이유를 설명할 수 있다.
- [ ] 원본 State와 파생값을 구분할 수 있다.
- [ ] 데이터 상태로 조건부 스타일과 입력 제한을 표현할 수 있다.

### 복습 질문 및 답변

**Q1. 완료 상태 변경에 `map`이 잘 맞는 이유는 무엇인가요?**

<details>
<summary>답</summary>

목록 길이와 순서를 유지하면서 대상 항목만 새 객체로 바꾼 새 배열을 만들 수 있기 때문입니다.

</details>

**Q2. 삭제 동작에 인덱스보다 id가 유리한 이유는 무엇인가요?**

<details>
<summary>답</summary>

정렬이나 필터링으로 화면 순서가 바뀌어도 id는 같은 항목을 가리키므로 잘못된 대상을 수정할 위험이 줄어듭니다.

</details>

**Q3. `isLimitReached`를 별도 State로 저장하면 어떤 문제가 생길 수 있나요?**

<details>
<summary>답</summary>

목록이 바뀔 때마다 두 State를 정확히 함께 갱신해야 하므로 서로 불일치할 가능성이 생깁니다. 목록 길이에서 계산하면 항상 원본과 일치합니다.

</details>

## 한 줄 정리

> Todo 앱의 핵심은 폼·목록·콜백을 단방향 데이터 흐름으로 연결하고 모든 목록 변경을 새 값으로 표현하는 것입니다.
