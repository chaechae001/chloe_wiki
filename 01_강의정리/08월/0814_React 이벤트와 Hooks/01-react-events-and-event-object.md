# React 이벤트와 이벤트 객체

React 화면은 사용자의 클릭과 입력을 이벤트로 받아 다음 상태를 결정합니다. 이벤트를 제대로 이해하면 정적인 컴포넌트를 실제로 반응하는 UI로 바꿀 수 있습니다.

`event handler` · `onClick` · `onChange` · `event.target` · `function reference`

## 핵심요약

- JSX 이벤트 속성에는 함수를 실행한 결과가 아니라 실행할 함수 자체를 전달합니다.
- 이벤트 객체에는 어떤 요소에서 어떤 상호작용이 일어났는지에 관한 정보가 들어 있습니다.
- 입력 요소의 현재값은 보통 `event.target.value`로 읽습니다.
- 짧은 동작은 인라인 함수로, 복잡하거나 재사용할 동작은 이름 있는 핸들러로 분리합니다.
- 이벤트 이름은 `on...`, 핸들러 이름은 `handle...` 형태로 쓰면 의도가 선명해집니다.

## 1. JSX에서 이벤트 연결하기

React 이벤트 속성은 camelCase로 작성합니다. `onclick`이 아니라 `onClick`, `onchange`가 아니라 `onChange`를 사용합니다.

```jsx
function ActionButton() {
  const handleClick = () => {
    console.log("저장 요청");
  };

  return <button onClick={handleClick}>저장</button>;
}
```

`onClick={handleClick}`은 클릭 시 실행할 함수를 등록합니다. `onClick={handleClick()}`처럼 괄호를 붙이면 렌더링 중 즉시 호출되므로 대부분 의도와 다릅니다.

짧고 한 번만 쓰는 로직은 다음처럼 인라인 함수로 표현할 수 있습니다.

```jsx
<button onClick={() => console.log("미리보기")}>미리보기</button>
```

처리 내용이 길어지거나 테스트해야 한다면 별도 함수로 분리하는 편이 읽기 쉽습니다.

## 2. 이벤트 객체로 입력 읽기

React는 DOM 이벤트 핸들러에 이벤트 객체를 전달합니다. `target`은 실제 이벤트가 발생한 요소이며, 입력 요소에서는 `value`로 현재 내용을 읽을 수 있습니다.

```jsx
function SearchBox() {
  const handleChange = (event) => {
    console.log(event.target.value);
  };

  return <input onChange={handleChange} placeholder="검색어" />;
}
```

클릭, 키보드, 포커스, 제출 이벤트마다 제공되는 정보가 다릅니다. 필요한 값을 무작정 외우기보다 “어느 요소에서 어떤 사건이 일어났는가”를 먼저 확인하는 습관이 중요합니다.

## 코드로 보기 — 버튼과 입력 이벤트 분리하기

```jsx
function Feedback() {
  const handleInput = (event) => {
    console.log(`현재 길이: ${event.target.value.length}`);
  };

  const handleSend = () => {
    console.log("피드백 전송");
  };

  return (
    <section>
      <input onChange={handleInput} />
      <button onClick={handleSend}>보내기</button>
    </section>
  );
}
```

### 코드 목적

입력 변경과 버튼 클릭이라는 서로 다른 사건을 각각의 핸들러로 처리합니다.

### 코드 흐름

1. 사용자가 입력하면 `handleInput`이 이벤트 객체를 받습니다.
2. `target.value`에서 최신 문자열을 읽습니다.
3. 버튼을 클릭하면 `handleSend`가 독립적으로 실행됩니다.

### 실행 결과 해석

입력할 때마다 문자열 길이가 콘솔에 표시되고, 버튼 클릭 시 전송 메시지가 출력됩니다. 각 핸들러가 한 가지 책임만 갖기 때문에 수정 범위도 작아집니다.

### 실무 연결

검색창, 회원가입 폼, 필터 버튼, 모달 닫기처럼 사용자의 행동을 애플리케이션 로직으로 연결할 때 같은 구조를 사용합니다.

## 직접 해보기

1. 버튼 클릭 시 콘솔에 `좋아요`를 출력하는 이름 있는 핸들러를 작성해 보세요.
2. 입력값에서 앞뒤 공백을 제거한 문자열을 콘솔에 출력해 보세요.
3. 입력 길이가 20자를 넘으면 경고 메시지를 출력하도록 확장해 보세요.

<details>
<summary>정답 보기</summary>

1. `const handleLike = () => console.log("좋아요")`를 만들고 `onClick={handleLike}`로 전달합니다.
2. `onChange` 핸들러에서 `event.target.value.trim()`을 출력합니다.
3. 최신 입력값의 `length`가 20보다 큰지 검사하고 조건을 만족할 때만 경고를 출력합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `onClick={handleClick}` vs `onClick={handleClick()}` | 앞은 함수를 등록하고, 뒤는 렌더링 중 함수를 즉시 실행합니다. |
| 이름 있는 함수 vs 인라인 함수 | 재사용·복잡한 로직에는 이름 있는 함수가 좋고, 짧은 일회성 로직에는 인라인 함수가 간결합니다. |
| `target` vs `value` | `target`은 이벤트가 발생한 요소이고, `value`는 입력 요소가 가진 현재값입니다. |

## 연결되는 개념

- 입력값을 화면 상태로 연결하려면 [폼과 제어 입력](02-forms-and-controlled-inputs.md)을 확인하세요.
- 자식의 동작을 부모에 알리는 방법은 [콜백 Props와 컴포넌트 이벤트](03-callback-props-and-component-events.md)에서 이어집니다.
- 관련 용어는 [용어집](GLOSSARY.md)에서 찾을 수 있습니다.

## 셀프 체크

- [ ] JSX 이벤트 속성의 표기법을 설명할 수 있다.
- [ ] 함수 참조와 함수 호출을 구분할 수 있다.
- [ ] 이벤트 객체에서 입력값을 읽을 수 있다.
- [ ] 인라인 함수와 이름 있는 핸들러를 상황에 맞게 고를 수 있다.
- [ ] `on...`과 `handle...` 명명 규칙을 적용할 수 있다.

### 복습 질문 및 답변

**Q1. `onClick`에 함수를 전달할 때 괄호를 보통 붙이지 않는 이유는 무엇인가요?**

<details>
<summary>답</summary>

괄호를 붙이면 클릭을 기다리지 않고 렌더링 중 함수가 실행되기 때문입니다. 이벤트 속성에는 나중에 실행할 함수 자체를 전달해야 합니다.

</details>

**Q2. 입력 요소의 최신 문자열은 어디에서 읽나요?**

<details>
<summary>답</summary>

이벤트 핸들러가 받은 객체의 `event.target.value`에서 읽습니다.

</details>

**Q3. 모든 이벤트를 인라인 함수로 작성하면 어떤 문제가 생길 수 있나요?**

<details>
<summary>답</summary>

로직이 길어질수록 JSX가 복잡해지고 재사용·테스트·디버깅이 어려워집니다. 복잡한 동작은 이름 있는 핸들러로 분리하는 편이 좋습니다.

</details>

## 한 줄 정리

> React 이벤트 처리는 사용자의 행동을 함수에 연결하고, 이벤트 객체에서 필요한 정보를 꺼내 다음 UI 상태를 만드는 과정입니다.
