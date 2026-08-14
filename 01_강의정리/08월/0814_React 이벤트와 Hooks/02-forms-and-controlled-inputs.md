# 폼과 제어 입력

폼은 사용자의 입력을 애플리케이션 데이터로 바꾸는 경계입니다. React에서는 입력값을 State와 연결하면 화면과 데이터의 기준을 하나로 유지할 수 있습니다.

`form` · `controlled input` · `onSubmit` · `preventDefault` · `computed property`

## 핵심요약

- 제어 입력은 `value`를 State에 연결하고 `onChange`로 State를 갱신합니다.
- 폼 제출은 버튼 클릭뿐 아니라 Enter 입력도 함께 처리할 수 있습니다.
- `event.preventDefault()`는 브라우저의 기본 제출·새로고침 동작을 막습니다.
- 여러 입력은 `name` 속성과 계산된 프로퍼티를 이용해 하나의 핸들러로 관리할 수 있습니다.
- 객체 State는 기존 객체를 직접 바꾸지 않고 새 객체를 만들어 갱신합니다.

## 1. 입력값을 State에 연결하기

입력 요소의 `value`를 State에 연결하면 화면에 보이는 값의 기준이 React State가 됩니다. 입력 이벤트에서 State를 갱신하지 않으면 사용자가 타이핑해도 값이 바뀌지 않습니다.

```jsx
import { useState } from "react";

function NameField() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={(event) => setName(event.target.value)}
    />
  );
}
```

이 구조를 제어 입력이라고 합니다. 검증, 글자 수 제한, 입력값 초기화처럼 값의 변화를 애플리케이션에서 일관되게 다루기 쉽습니다.

## 2. 폼 제출과 기본 동작 제어

`form`의 `onSubmit`에 처리 함수를 연결하면 제출 버튼과 Enter 키를 같은 흐름으로 처리할 수 있습니다.

```jsx
function KeywordForm({ onSearch }) {
  const [keyword, setKeyword] = useState("");

  const handleSubmit = (event) => {
    event.preventDefault();
    const value = keyword.trim();

    if (value) {
      onSearch(value);
      setKeyword("");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={keyword} onChange={(e) => setKeyword(e.target.value)} />
      <button type="submit">검색</button>
    </form>
  );
}
```

`preventDefault()`가 없으면 브라우저가 문서 전송과 새로고침을 시도할 수 있습니다. SPA에서는 JavaScript가 제출 흐름을 맡는 경우가 많으므로 기본 동작을 막고 필요한 로직만 실행합니다.

## 3. 여러 입력을 하나의 핸들러로 관리하기

입력의 `name`을 State 객체의 키와 맞추면 하나의 핸들러로 여러 필드를 처리할 수 있습니다.

```jsx
const [profile, setProfile] = useState({ nickname: "", city: "" });

const handleChange = (event) => {
  const { name, value } = event.target;
  setProfile((current) => ({
    ...current,
    [name]: value,
  }));
};
```

`[name]`은 변수에 담긴 문자열을 객체 키로 사용하는 계산된 프로퍼티입니다. 나머지 속성은 전개 문법으로 복사하므로 한 필드만 바꿔도 다른 값이 유지됩니다.

## 코드로 보기 — 두 필드 프로필 폼

```jsx
import { useState } from "react";

function ProfileForm() {
  const [profile, setProfile] = useState({ nickname: "", role: "" });

  const handleChange = ({ target: { name, value } }) => {
    setProfile((current) => ({ ...current, [name]: value }));
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    console.log(`${profile.nickname}: ${profile.role}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="nickname" value={profile.nickname} onChange={handleChange} />
      <input name="role" value={profile.role} onChange={handleChange} />
      <button type="submit">등록</button>
    </form>
  );
}
```

### 코드 목적

두 개의 입력을 하나의 객체 State와 하나의 변경 핸들러로 관리합니다.

### 코드 흐름

1. 각 입력의 `name`이 바꿀 State 키를 알려 줍니다.
2. `handleChange`가 현재 객체를 복사하고 해당 키만 덮어씁니다.
3. 제출 시 기본 동작을 막고 최신 State를 사용합니다.

### 실행 결과 해석

한 입력을 수정해도 다른 입력값이 사라지지 않습니다. 제출 시 두 필드의 최신값이 한 번에 처리됩니다.

### 실무 연결

회원가입, 검색 조건, 배송지, 관리자 설정처럼 여러 입력을 함께 검증하고 전송하는 화면에서 활용됩니다.

## 직접 해보기

1. 이메일 입력을 제어 입력으로 만들고 화면 아래에 현재값을 출력해 보세요.
2. 제출 시 빈 문자열이면 처리하지 않도록 조건을 추가해 보세요.
3. `title`, `category`, `priority` 세 입력을 하나의 객체 State로 관리해 보세요.

<details>
<summary>정답 보기</summary>

1. `value={email}`과 `onChange={(e) => setEmail(e.target.value)}`를 연결하고 JSX에 `{email}`을 출력합니다.
2. `event.preventDefault()` 뒤에서 `value.trim()`이 빈 문자열인지 검사합니다.
3. 각 입력의 `name`을 객체 키와 맞추고 `setState(current => ({ ...current, [name]: value }))` 형태로 갱신합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `value` vs `defaultValue` | `value`는 현재값을 State가 관리하고, `defaultValue`는 DOM 입력의 초기값만 정합니다. |
| `onClick` vs `onSubmit` | 클릭은 특정 버튼 동작이고, 제출은 버튼과 Enter를 포함한 폼 전체 동작입니다. |
| `profile.name` vs `profile[name]` | 앞은 고정 키를, 뒤는 변수에 담긴 동적 키를 사용합니다. |

## 연결되는 개념

- 이벤트 객체의 기본은 [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md)에서 확인하세요.
- 제출 데이터를 부모로 전달하는 구조는 [콜백 Props와 컴포넌트 이벤트](03-callback-props-and-component-events.md)에서 이어집니다.
- 객체 갱신 원리는 [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md)에서도 사용합니다.

## 셀프 체크

- [ ] 제어 입력의 데이터 흐름을 설명할 수 있다.
- [ ] `preventDefault()`가 필요한 이유를 말할 수 있다.
- [ ] 여러 입력을 하나의 핸들러로 처리할 수 있다.
- [ ] 객체 State를 불변하게 갱신할 수 있다.
- [ ] 입력값을 검증하고 제출 후 초기화할 수 있다.

### 복습 질문 및 답변

**Q1. 제어 입력에서 `onChange`가 빠지면 어떻게 되나요?**

<details>
<summary>답</summary>

`value`가 State로 고정되어 있는데 State를 바꿀 경로가 없으므로 사용자가 입력해도 화면 값이 갱신되지 않습니다.

</details>

**Q2. 여러 입력을 하나의 핸들러로 구분하는 기준은 무엇인가요?**

<details>
<summary>답</summary>

각 입력의 `name` 속성을 State 객체의 키와 맞추고, 이벤트 객체에서 `name`과 `value`를 함께 읽습니다.

</details>

**Q3. 객체 State의 특정 속성만 직접 바꾸면 왜 위험한가요?**

<details>
<summary>답</summary>

기존 객체의 참조가 유지되면 React가 변경을 안정적으로 감지하기 어렵고 이전 상태도 오염될 수 있습니다. 새 객체를 반환해야 변경 경계가 명확합니다.

</details>

## 한 줄 정리

> React 폼은 입력값을 State로 제어하고, 제출 이벤트에서 검증·전달·초기화를 한 흐름으로 묶습니다.
