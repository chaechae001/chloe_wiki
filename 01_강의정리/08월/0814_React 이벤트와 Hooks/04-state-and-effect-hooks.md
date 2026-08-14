# State Hook과 Effect Hook

Hook은 함수 컴포넌트가 렌더링 사이의 값을 기억하고 외부 시스템과 동기화하도록 돕습니다. 그중 `useState`와 `useEffect`는 대부분의 상호작용을 만드는 출발점입니다.

`Hook rules` · `useState` · `useEffect` · `dependency array` · `cleanup`

## 핵심요약

- Hook은 함수 컴포넌트나 다른 Hook의 최상위에서 호출합니다.
- `useState`는 렌더링 사이에 유지할 값을 선언하고 setter로 다음 상태를 요청합니다.
- 이전 상태를 기준으로 갱신할 때는 함수형 갱신을 사용합니다.
- `useEffect`는 렌더링 결과를 외부 시스템과 동기화하는 데 사용합니다.
- Effect는 의존성이 바뀌기 전과 컴포넌트가 사라질 때 정리 함수를 실행할 수 있습니다.

## 1. Hook 호출 규칙

React는 Hook 호출 순서로 각 State와 Effect를 구분합니다. 따라서 조건문, 반복문, 중첩 함수 안에서 Hook을 호출하면 렌더링마다 순서가 달라질 수 있습니다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  if (count > 10) {
    // 이 안에서 새로운 Hook을 호출하면 안 됩니다.
  }

  return <button onClick={() => setCount((current) => current + 1)}>{count}</button>;
}
```

Hook 이름은 `use`로 시작합니다. 이 규칙은 React와 린터가 Hook 사용을 식별하는 데도 도움이 됩니다.

## 2. 직접 갱신과 함수형 갱신

다음 상태가 이전 상태에 의존하지 않으면 값을 직접 전달해도 됩니다. 이전 값을 바탕으로 계산한다면 updater 함수를 사용합니다.

```jsx
setTitle("완료");
setCount((current) => current + 1);
```

React는 여러 상태 변경을 모아 처리할 수 있습니다. 함수형 갱신은 각 업데이트가 가장 최신의 이전 상태를 기준으로 계산되도록 합니다.

## 3. Effect와 의존성 배열

Effect는 렌더링 자체가 아니라 렌더링으로 생긴 결과를 브라우저 API, 타이머, 네트워크, 구독 같은 외부 시스템과 맞출 때 사용합니다.

```jsx
useEffect(() => {
  document.title = `알림 ${count}개`;
}, [count]);
```

- 의존성 배열을 생략하면 매 렌더링 후 실행됩니다.
- 빈 배열이면 마운트 후 실행되고 언마운트 시 정리됩니다.
- `[count]`처럼 값을 넣으면 해당 값이 달라졌을 때 다시 동기화합니다.

개발 환경의 Strict Mode에서는 문제 있는 Effect를 찾기 위해 설정과 정리를 추가로 실행할 수 있습니다. “무조건 한 번”이라고 외우기보다 설정과 정리가 서로 대칭인지 확인하는 편이 안전합니다.

## 코드로 보기 — 타이머 시작과 정리

```jsx
import { useEffect, useState } from "react";

function Clock() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const timerId = setInterval(() => {
      setSeconds((current) => current + 1);
    }, 1000);

    return () => clearInterval(timerId);
  }, []);

  return <p>실행 시간: {seconds}초</p>;
}
```

### 코드 목적

컴포넌트가 화면에 있는 동안만 타이머를 실행하고 사라질 때 자원을 해제합니다.

### 코드 흐름

1. 첫 렌더링 후 interval을 등록합니다.
2. 매초 함수형 갱신으로 State를 증가시킵니다.
3. 컴포넌트가 사라지기 전 cleanup이 interval을 해제합니다.

### 실행 결과 해석

화면 숫자는 매초 증가합니다. 정리 함수가 없다면 화면에서 제거된 뒤에도 타이머가 남아 불필요한 작업을 계속할 수 있습니다.

### 실무 연결

실시간 연결, 이벤트 구독, 타이머, 브라우저 API처럼 시작과 종료가 한 쌍인 작업에 Effect와 cleanup을 사용합니다.

## 직접 해보기

1. 버튼 클릭마다 이전값을 기준으로 카운트를 2씩 증가시켜 보세요.
2. 검색어 State가 바뀔 때마다 문서 제목을 갱신하는 Effect를 작성해 보세요.
3. 창 크기 변경 이벤트를 등록하고 cleanup에서 해제하는 Effect를 설계해 보세요.

<details>
<summary>정답 보기</summary>

1. `setCount(current => current + 2)`를 사용합니다.
2. `useEffect(() => { document.title = keyword; }, [keyword])`처럼 검색어를 의존성에 둡니다.
3. Effect에서 `window.addEventListener("resize", handler)`를 호출하고 반환 함수에서 같은 handler로 `removeEventListener`를 호출합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 렌더링 계산 vs Effect | JSX와 파생값 계산은 렌더링에서, 외부 시스템 동기화는 Effect에서 수행합니다. |
| 값 갱신 vs 함수형 갱신 | 고정된 다음값은 직접 전달하고, 이전값 기반 계산은 updater 함수를 사용합니다. |
| 빈 의존성 배열 vs 배열 생략 | 빈 배열은 설정·정리 생명주기에 맞고, 생략하면 매 렌더링 후 Effect가 실행됩니다. |

## 연결되는 개념

- Hook을 사용한 성능·DOM 제어는 [메모이제이션과 ref](05-memoization-and-refs.md)에서 이어집니다.
- 반복 상태 로직을 분리하는 방법은 [커스텀 Hook 설계](06-custom-hooks.md)에서 설명합니다.
- 이벤트에서 State를 바꾸는 흐름은 [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md)를 확인하세요.

## 셀프 체크

- [ ] Hook의 두 가지 핵심 호출 규칙을 설명할 수 있다.
- [ ] 함수형 갱신이 필요한 상황을 구분할 수 있다.
- [ ] Effect가 필요한 작업과 불필요한 작업을 구분할 수 있다.
- [ ] 의존성 배열의 의미를 설명할 수 있다.
- [ ] 설정과 cleanup을 한 쌍으로 작성할 수 있다.

### 복습 질문 및 답변

**Q1. Hook을 조건문 안에서 호출하면 안 되는 이유는 무엇인가요?**

<details>
<summary>답</summary>

렌더링마다 Hook 호출 순서가 달라질 수 있고, React가 어떤 State와 Effect가 어느 호출에 대응하는지 안정적으로 구분할 수 없기 때문입니다.

</details>

**Q2. `useEffect`의 cleanup은 언제 필요한가요?**

<details>
<summary>답</summary>

타이머, 구독, 이벤트 리스너처럼 Effect가 시작한 외부 작업을 의존성 변경 전이나 언마운트 시 해제해야 할 때 필요합니다.

</details>

**Q3. 이전 State를 기반으로 연속 갱신할 때 updater 함수가 안전한 이유는 무엇인가요?**

<details>
<summary>답</summary>

React가 처리 순서에 맞는 최신 이전값을 updater에 전달하므로, 렌더링 시점에 캡처된 오래된 값에 의존하는 문제를 줄일 수 있습니다.

</details>

## 한 줄 정리

> `useState`는 UI가 기억할 값을, `useEffect`는 그 렌더링 결과와 외부 시스템의 동기화를 담당합니다.
