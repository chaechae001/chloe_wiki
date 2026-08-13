# Props와 단방향 데이터 흐름

Props는 부모가 자식 컴포넌트에 전달하는 읽기 전용 입력으로, 같은 UI 구조를 서로 다른 데이터에 재사용하게 합니다.

## 핵심 키워드

`Props` · `read-only` · `destructuring` · `one-way data flow` · `callback`

## 핵심 요약

- Props는 컴포넌트 바깥에서 전달되는 입력입니다.
- 자식은 전달받은 Props를 직접 변경하지 않습니다.
- 구조 분해로 필요한 Props를 명확히 꺼낼 수 있습니다.
- 데이터는 부모에서 자식으로 흐르고 변경 요청은 콜백으로 올릴 수 있습니다.

## 1. Props 전달과 사용

```jsx
function Welcome({ name, role = "회원" }) {
  return <p>{name}님, {role} 화면에 오신 것을 환영합니다.</p>;
}

function App() {
  return <Welcome name="새싹" role="관리자" />;
}
```

문자열 리터럴은 따옴표로, 숫자·객체·함수 같은 JavaScript 값은 중괄호로 전달합니다.

## 2. Props는 읽기 전용

자식이 Props를 직접 수정하면 부모가 가진 데이터와 화면의 기준이 불명확해집니다. 가공이 필요하면 새 변수나 파생값을 만듭니다.

```jsx
function Price({ amount }) {
  const formatted = amount.toLocaleString();
  return <strong>{formatted}원</strong>;
}
```

## 3. 객체와 함수 전달

```jsx
function UserCard({ user, onSelect }) {
  return (
    <button onClick={() => onSelect(user.id)}>
      {user.name} 선택
    </button>
  );
}
```

부모가 객체와 이벤트 처리 함수를 전달하면 자식은 표시와 사용자 입력 전달에 집중할 수 있습니다.

## 대표 코드: 부모가 선택 상태 관리하기

### 목적

부모가 데이터를 소유하고 자식이 콜백으로 변경 의도를 전달합니다.

```jsx
import { useState } from "react";

function Choice({ value, selected, onChoose }) {
  return (
    <button aria-pressed={selected} onClick={() => onChoose(value)}>
      {value}
    </button>
  );
}

function ChoiceGroup() {
  const [choice, setChoice] = useState("A");

  return (
    <div>
      {["A", "B"].map((value) => (
        <Choice key={value} value={value} selected={choice === value} onChoose={setChoice} />
      ))}
    </div>
  );
}
```

### 코드 흐름과 결과

1. 부모가 선택 State를 소유합니다.
2. 각 자식에 현재 값과 선택 여부를 전달합니다.
3. 자식 클릭이 부모 콜백을 호출합니다.
4. 부모 State가 바뀌며 두 자식의 Props가 다시 계산됩니다.

### 실무 연결

폼 필드, 선택 카드, 테이블 행처럼 표시 컴포넌트와 상태 관리 컴포넌트를 분리할 때 사용합니다.

## 직접 해보기

1. Props가 읽기 전용인 이유를 설명하세요.
2. 객체 Props에서 `title`과 `count`를 구조 분해하세요.
3. 자식이 부모 데이터를 변경해야 할 때의 패턴을 설명하세요.

<details>
<summary>정답 보기</summary>

1. 데이터 소유자와 변경 책임을 분명히 하여 예측 가능한 단방향 흐름을 유지하기 위해서입니다.
2. `function Summary({ info: { title, count } }) { ... }`처럼 꺼낼 수 있습니다.
3. 부모가 변경 함수를 Props로 전달하고 자식이 필요한 값과 함께 호출합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| Props vs State | Props는 외부 입력, State는 컴포넌트가 기억하고 setter로 갱신하는 값입니다. |
| Props 가공 vs Props 변경 | 새 변수를 만드는 것은 괜찮지만 전달받은 객체를 직접 수정하면 안 됩니다. |
| 함수 전달 vs 함수 호출 | `onClick={handler}`는 전달, `onClick={handler()}`는 렌더 중 즉시 호출입니다. |

## 연결되는 개념

- Props를 받는 단위는 [컴포넌트와 children](03-components-children-and-rendering.md)에서 설명합니다.
- 내부 데이터 변경은 [State와 불변 갱신](05-state-and-immutable-updates.md)에서 이어집니다.
- 목록 전달은 [목록 key와 입력 상태 설계](06-list-keys-and-form-state.md)에서 다룹니다.

## 셀프 체크

- [ ] 여러 타입의 Props를 전달할 수 있다.
- [ ] Props를 직접 변경하지 않는 이유를 안다.
- [ ] 콜백 Props로 변경 의도를 전달할 수 있다.

## 복습 질문 및 답변

### Q1. Props의 데이터 흐름 방향은?

<details>
<summary>답</summary>

기본적으로 부모 컴포넌트에서 자식 컴포넌트 방향입니다.

</details>

### Q2. Props에 기본값을 줄 수 있는가?

<details>
<summary>답</summary>

구조 분해 기본값 등으로 값이 undefined일 때 사용할 기본값을 정할 수 있습니다.

</details>

### Q3. 객체 Props를 자식에서 수정하면 왜 위험한가?

<details>
<summary>답</summary>

부모와 같은 객체 참조를 공유할 수 있어 부모 데이터까지 예기치 않게 바뀌고 변화 추적이 어려워집니다.

</details>

## 한 줄 정리

> Props는 부모가 소유한 데이터를 자식에 읽기 전용으로 전달해 재사용과 단방향 흐름을 만드는 입력입니다.
