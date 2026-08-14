# 콜백 Props와 컴포넌트 이벤트

자식 컴포넌트는 부모의 State를 직접 수정하지 않습니다. 대신 부모가 전달한 콜백을 호출해 “어떤 일이 일어났다”는 사실과 필요한 데이터를 알립니다.

`callback props` · `data flow` · `custom event` · `lifting state` · `on...`

## 핵심요약

- 데이터는 Props를 통해 부모에서 자식으로 내려갑니다.
- 자식의 행동은 콜백 Props를 호출해 부모로 알립니다.
- 부모는 State와 변경 로직을 소유하고 자식은 사용자 입력을 전달하는 역할을 맡을 수 있습니다.
- 컴포넌트 이벤트 이름은 `onSave`, `onRemove`처럼 의미 있는 동작으로 표현합니다.
- 콜백을 호출할 때는 부모가 판단하는 데 필요한 최소 데이터만 전달합니다.

## 1. 부모가 함수를 내려주는 이유

공유 데이터는 가장 가까운 공통 부모가 소유하는 편이 흐름을 추적하기 쉽습니다. 자식은 부모의 함수를 Props로 받아 DOM 이벤트에 연결합니다.

```jsx
function TextField({ onTextChange }) {
  return <input onChange={(event) => onTextChange(event.target.value)} />;
}

function App() {
  const [text, setText] = useState("");

  return (
    <>
      <h2>{text}</h2>
      <TextField onTextChange={setText} />
    </>
  );
}
```

사용자 입력은 자식에서 발생하지만 실제 데이터는 부모가 소유합니다. 이 구조 덕분에 형제 컴포넌트도 같은 State를 받아 사용할 수 있습니다.

## 2. DOM 이벤트와 컴포넌트 이벤트

`onClick`과 `onChange`는 DOM 요소가 제공하는 이벤트입니다. 반면 `onSave`, `onComplete`, `onCancel` 같은 이름은 컴포넌트의 의미에 맞게 직접 정한 콜백 Props입니다.

```jsx
function SaveButton({ onSave }) {
  return <button onClick={() => onSave({ savedAt: Date.now() })}>저장</button>;
}
```

컴포넌트 내부에서는 클릭이라는 세부 구현을 처리하고, 외부에는 저장이라는 의미 있는 동작을 공개합니다. 나중에 클릭 대신 단축키로 저장하게 바뀌어도 부모 인터페이스는 유지할 수 있습니다.

## 코드로 보기 — 항목 추가 요청 전달하기

```jsx
import { useState } from "react";

function AddItemForm({ onAdd }) {
  const [value, setValue] = useState("");

  const handleSubmit = (event) => {
    event.preventDefault();
    const nextValue = value.trim();

    if (nextValue) {
      onAdd(nextValue);
      setValue("");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <button type="submit">추가</button>
    </form>
  );
}

function App() {
  const [items, setItems] = useState([]);

  return <AddItemForm onAdd={(value) => setItems((list) => [...list, value])} />;
}
```

### 코드 목적

입력 UI와 목록 State의 소유권을 분리하면서 자식의 제출 결과를 부모에 전달합니다.

### 코드 흐름

1. 자식이 입력값을 임시 State로 관리합니다.
2. 제출 시 빈 값을 제거하고 `onAdd`를 호출합니다.
3. 부모가 받은 값을 자신의 목록 State에 추가합니다.

### 실행 결과 해석

자식은 목록의 저장 방식이나 다른 항목을 알 필요가 없습니다. 부모는 입력 요소의 세부 동작을 알지 않아도 새 값만 받아 목록을 갱신합니다.

### 실무 연결

모달의 확인·취소, 리스트 항목의 수정·삭제, 파일 선택 완료처럼 하위 UI의 결과를 상위 화면에 전달하는 데 사용됩니다.

## 직접 해보기

1. 자식 버튼이 클릭되면 부모에서 카운트를 1 증가시키는 콜백 Props를 작성해 보세요.
2. `onSelect`에 항목의 `id`만 전달하는 목록 컴포넌트를 만들어 보세요.
3. 모달 컴포넌트가 `onConfirm`과 `onCancel`을 각각 호출하도록 설계해 보세요.

<details>
<summary>정답 보기</summary>

1. 부모가 `handleIncrease`를 만들고 자식에 `onIncrease`로 전달한 뒤 자식 버튼의 `onClick`에서 호출합니다.
2. `items.map(item => <button onClick={() => onSelect(item.id)}>...</button>)`처럼 필요한 식별자만 넘깁니다.
3. 확인 버튼에는 `onConfirm`, 취소 버튼에는 `onCancel`을 연결하고 실제 State 변경은 부모에서 수행합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 값 Props vs 콜백 Props | 값 Props는 데이터를 내려주고, 콜백 Props는 자식의 행동을 부모에 알립니다. |
| DOM 이벤트 vs 컴포넌트 이벤트 | DOM 이벤트는 브라우저 사건이고, 컴포넌트 이벤트는 도메인 의미를 담은 사용자 정의 인터페이스입니다. |
| 함수 전달 vs 함수 실행 | `onSave={handleSave}`는 전달이고, `onSave={handleSave()}`는 렌더링 중 실행입니다. |

## 연결되는 개념

- DOM 이벤트의 기본은 [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md)를 확인하세요.
- 자식 폼의 입력 관리는 [폼과 제어 입력](02-forms-and-controlled-inputs.md)에서 설명합니다.
- 여러 콜백을 조합하는 예시는 [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md)에서 이어집니다.

## 셀프 체크

- [ ] 콜백 Props가 필요한 이유를 설명할 수 있다.
- [ ] 데이터 하향·이벤트 상향 흐름을 그릴 수 있다.
- [ ] 컴포넌트 의미에 맞는 이벤트 이름을 정할 수 있다.
- [ ] 콜백에 필요한 최소 데이터만 전달할 수 있다.
- [ ] 부모와 자식의 책임을 나눌 수 있다.

### 복습 질문 및 답변

**Q1. 자식 컴포넌트가 부모 State를 직접 바꾸지 않는 이유는 무엇인가요?**

<details>
<summary>답</summary>

State의 소유자와 변경 지점을 한곳에 두면 데이터 흐름이 예측 가능하고 디버깅하기 쉽기 때문입니다.

</details>

**Q2. `onClick`을 그대로 노출하지 않고 `onSave` 같은 이름을 쓰는 장점은 무엇인가요?**

<details>
<summary>답</summary>

외부 컴포넌트는 구현 수단보다 동작의 의미를 이해하게 됩니다. 내부 구현이 바뀌어도 인터페이스를 유지하기 쉽습니다.

</details>

**Q3. 목록 항목 전체 대신 `id`만 콜백에 전달하면 어떤 장점이 있나요?**

<details>
<summary>답</summary>

컴포넌트 사이 결합이 줄고, 부모가 최신 State에서 해당 항목을 찾아 일관된 방식으로 처리할 수 있습니다.

</details>

## 한 줄 정리

> 콜백 Props는 자식의 사용자 행동을 부모가 소유한 State 변경으로 안전하게 연결하는 컴포넌트 인터페이스입니다.
