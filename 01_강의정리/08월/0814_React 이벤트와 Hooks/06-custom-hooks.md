# 커스텀 Hook 설계

컴포넌트를 재사용하면 UI 구조를 공유할 수 있고, 커스텀 Hook을 재사용하면 상태를 다루는 규칙을 공유할 수 있습니다. 핵심은 State 자체가 아니라 상태 로직을 추출하는 것입니다.

`custom Hook` · `use prefix` · `logic reuse` · `encapsulation` · `independent state`

## 핵심요약

- 커스텀 Hook은 React Hook을 조합한 일반 JavaScript 함수입니다.
- 이름은 `use`로 시작하고 Hook 호출 규칙을 지켜야 합니다.
- 같은 커스텀 Hook을 여러 컴포넌트에서 호출해도 각 호출의 State는 독립적입니다.
- 반환값은 사용자가 필요한 상태와 동작만 드러내도록 설계합니다.
- 반복되는 로직이 실제로 생겼을 때 추출하면 인터페이스를 더 정확하게 정할 수 있습니다.

## 1. UI 재사용과 로직 재사용

컴포넌트는 JSX를 반환해 화면을 구성합니다. 커스텀 Hook은 JSX를 반환할 필요가 없으며, State·Effect·콜백 같은 로직을 묶어 반환합니다.

예를 들어 여러 화면에 켜짐·꺼짐 전환이 반복된다면 각 컴포넌트가 같은 `useState`와 토글 함수를 복제하지 않고 하나의 Hook으로 만들 수 있습니다.

## 2. 작은 인터페이스 만들기

```jsx
import { useCallback, useState } from "react";

export function useToggle(initialValue = false) {
  const [isOn, setIsOn] = useState(initialValue);

  const toggle = useCallback(() => {
    setIsOn((current) => !current);
  }, []);

  return { isOn, toggle };
}
```

사용하는 쪽은 State setter의 세부 구현을 몰라도 됩니다. `isOn`을 읽고 `toggle`을 호출한다는 작은 인터페이스만 이해하면 됩니다.

## 3. 각 호출의 State는 독립적이다

```jsx
function Settings() {
  const menu = useToggle();
  const notifications = useToggle(true);

  return (
    <>
      <button onClick={menu.toggle}>메뉴: {menu.isOn ? "열림" : "닫힘"}</button>
      <button onClick={notifications.toggle}>
        알림: {notifications.isOn ? "켜짐" : "꺼짐"}
      </button>
    </>
  );
}
```

두 호출은 같은 로직을 사용하지만 각각 별도의 State를 가집니다. 데이터를 공유하려면 공통 부모의 State, Context 또는 별도 상태 저장소 같은 다른 구조가 필요합니다.

## 코드로 보기 — 입력값 관리 Hook

```jsx
import { useState } from "react";

function useInput(initialValue = "") {
  const [value, setValue] = useState(initialValue);

  const bind = {
    value,
    onChange: (event) => setValue(event.target.value),
  };

  const reset = () => setValue(initialValue);

  return { value, bind, reset };
}

function NoteForm() {
  const title = useInput("");

  return (
    <form onSubmit={(event) => event.preventDefault()}>
      <input {...title.bind} />
      <button type="button" onClick={title.reset}>초기화</button>
      <p>{title.value}</p>
    </form>
  );
}
```

### 코드 목적

제어 입력의 값, 변경 이벤트, 초기화 로직을 하나의 재사용 가능한 Hook으로 묶습니다.

### 코드 흐름

1. Hook이 입력 State를 선언합니다.
2. `bind`가 입력에 필요한 `value`와 `onChange`를 제공합니다.
3. `reset`이 초기값으로 되돌립니다.
4. 컴포넌트는 반환된 인터페이스만 JSX에 연결합니다.

### 실행 결과 해석

입력할 때마다 `title.value`가 갱신되고 초기화 버튼은 입력을 처음 상태로 되돌립니다. 다른 컴포넌트에서 호출하면 동일한 규칙을 독립적으로 재사용합니다.

### 실무 연결

폼 필드, 토글, 로컬 저장소 동기화, 네트워크 요청 상태처럼 여러 화면에서 반복되는 상태 로직을 일관되게 관리할 수 있습니다.

## 직접 해보기

1. `useToggle(true)`를 호출해 초기 상태가 켜진 버튼을 만들어 보세요.
2. 숫자를 증가·감소·초기화하는 `useCounter`를 설계해 보세요.
3. `useInput`에 최대 글자 수를 받아 입력을 제한하는 기능을 추가해 보세요.

<details>
<summary>정답 보기</summary>

1. 반환된 `isOn`과 `toggle`을 버튼 텍스트와 `onClick`에 연결합니다.
2. 내부에 숫자 State를 두고 `increment`, `decrement`, `reset` 함수를 반환합니다. 이전값 기반 변경에는 함수형 갱신을 사용합니다.
3. `onChange`에서 새 문자열의 길이가 최대값 이하일 때만 `setValue`를 호출합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 컴포넌트 vs 커스텀 Hook | 컴포넌트는 UI를 구성하고, 커스텀 Hook은 상태 로직과 동작을 재사용합니다. |
| 로직 공유 vs State 공유 | 같은 Hook을 호출하면 로직은 공유하지만 각 호출의 State는 독립적입니다. |
| 일반 함수 vs 커스텀 Hook | 커스텀 Hook은 다른 Hook을 호출할 수 있고 이름과 호출 위치 규칙을 지켜야 합니다. |

## 연결되는 개념

- Hook 호출 규칙은 [State Hook과 Effect Hook](04-state-and-effect-hooks.md)에서 설명합니다.
- 콜백 참조를 보존하는 방법은 [메모이제이션과 ref](05-memoization-and-refs.md)에서 확인하세요.
- 커스텀 Hook을 사용할 UI 이벤트는 [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md)와 연결됩니다.

## 셀프 체크

- [ ] 커스텀 Hook의 목적을 설명할 수 있다.
- [ ] `use` 접두사와 최상위 호출 규칙을 지킬 수 있다.
- [ ] State 공유와 로직 공유를 구분할 수 있다.
- [ ] 반환 인터페이스를 작고 명확하게 설계할 수 있다.
- [ ] 반복 로직을 Hook으로 추출할 수 있다.

### 복습 질문 및 답변

**Q1. 커스텀 Hook의 이름이 `use`로 시작해야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

해당 함수가 Hook 규칙을 따르고 다른 Hook을 호출할 수 있음을 React 도구와 개발자에게 알려 주기 때문입니다.

</details>

**Q2. 같은 `useToggle`을 두 번 호출하면 상태가 함께 바뀌나요?**

<details>
<summary>답</summary>

아닙니다. 각 호출은 독립적인 State를 가집니다. 공유되는 것은 구현 로직입니다.

</details>

**Q3. 반복이 생기기 전에 모든 상태 로직을 Hook으로 만들면 어떤 단점이 있나요?**

<details>
<summary>답</summary>

필요한 인터페이스가 아직 분명하지 않아 불필요한 추상화가 생길 수 있습니다. 실제 반복과 변화 지점을 확인한 뒤 추출하는 편이 유지보수에 유리합니다.

</details>

## 한 줄 정리

> 커스텀 Hook은 반복되는 상태 처리 규칙을 작은 인터페이스로 감싸 여러 컴포넌트에서 독립적으로 재사용하게 합니다.
