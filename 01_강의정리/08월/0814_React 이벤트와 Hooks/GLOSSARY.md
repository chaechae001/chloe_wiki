# 용어집

React 이벤트 처리와 Hook 기반 상태 설계에서 자주 사용하는 용어를 쉬운 말로 정리했습니다.

## 이벤트와 폼

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 이벤트 | 클릭·입력·제출처럼 브라우저 요소에서 발생한 사건을 알리는 신호입니다. | [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md) | 이벤트 핸들러 |
| 이벤트 핸들러 | 이벤트가 발생했을 때 실행할 JavaScript 함수입니다. | [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md) | 함수 참조 |
| 이벤트 객체 | 사건의 종류와 대상 요소 등 이벤트 정보를 담아 핸들러에 전달되는 객체입니다. | [React 이벤트와 이벤트 객체](01-react-events-and-event-object.md) | `target`, `value` |
| 제어 입력 | 현재 입력값을 React State가 관리하는 입력 요소입니다. | [폼과 제어 입력](02-forms-and-controlled-inputs.md) | `value`, `onChange` |
| `preventDefault` | 폼 제출 후 새로고침 같은 브라우저의 기본 동작을 막는 메서드입니다. | [폼과 제어 입력](02-forms-and-controlled-inputs.md) | `onSubmit` |
| 계산된 프로퍼티 | 변수의 문자열 값을 객체 키로 사용하는 문법입니다. | [폼과 제어 입력](02-forms-and-controlled-inputs.md) | `[name]` |
| 콜백 Props | 자식이 부모에 동작과 데이터를 알릴 수 있도록 부모가 함수로 내려주는 Props입니다. | [콜백 Props와 컴포넌트 이벤트](03-callback-props-and-component-events.md) | 단방향 데이터 흐름 |
| 컴포넌트 이벤트 | `onSave`, `onRemove`처럼 컴포넌트 의미에 맞게 설계한 콜백 인터페이스입니다. | [콜백 Props와 컴포넌트 이벤트](03-callback-props-and-component-events.md) | DOM 이벤트 |

## Hook과 렌더링

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Hook | 함수 컴포넌트에서 State, Effect 등 React 기능을 사용하도록 하는 함수입니다. | [State Hook과 Effect Hook](04-state-and-effect-hooks.md) | Hook 규칙 |
| 함수형 갱신 | setter에 이전 State를 받는 함수를 전달해 다음 State를 계산하는 방식입니다. | [State Hook과 Effect Hook](04-state-and-effect-hooks.md) | updater 함수 |
| Effect | 렌더링 결과를 브라우저 API나 외부 시스템과 동기화하는 작업입니다. | [State Hook과 Effect Hook](04-state-and-effect-hooks.md) | side effect |
| 의존성 배열 | Effect나 메모이제이션이 어떤 값의 변화에 반응할지 나타내는 배열입니다. | [State Hook과 Effect Hook](04-state-and-effect-hooks.md) | cleanup |
| cleanup | Effect가 등록한 타이머·구독·리스너 등을 다음 실행 전이나 언마운트 시 해제하는 함수입니다. | [State Hook과 Effect Hook](04-state-and-effect-hooks.md) | 생명주기 |
| 메모이제이션 | 같은 입력일 때 이전 계산값이나 참조를 재사용해 반복 작업을 줄이는 기법입니다. | [메모이제이션과 ref](05-memoization-and-refs.md) | `useMemo` |
| `useMemo` | 의존성이 같을 때 계산 결과를 재사용하는 Hook입니다. | [메모이제이션과 ref](05-memoization-and-refs.md) | 파생값 |
| `useCallback` | 의존성이 같을 때 함수 참조를 재사용하는 Hook입니다. | [메모이제이션과 ref](05-memoization-and-refs.md) | 함수 동일성 |
| `useRef` | 렌더링 사이에 값을 보존하되 `.current` 변경만으로 재렌더링하지 않는 ref 객체를 만드는 Hook입니다. | [메모이제이션과 ref](05-memoization-and-refs.md) | DOM 참조 |
| 커스텀 Hook | 여러 Hook과 상태 처리 규칙을 재사용 가능한 함수로 묶은 것입니다. | [커스텀 Hook 설계](06-custom-hooks.md) | 로직 재사용 |

## 목록과 상태 설계

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 불변 갱신 | 기존 배열이나 객체를 직접 수정하지 않고 변경을 반영한 새 값을 만드는 방식입니다. | [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md) | 참조 동일성 |
| `key` | React가 형제 목록 항목의 정체성을 렌더링 사이에 비교하도록 돕는 값입니다. | [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md) | 안정적인 id |
| 파생값 | 원본 State에서 계산할 수 있어 별도 State로 중복 저장할 필요가 없는 값입니다. | [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md) | 단일 진실 공급원 |
| 조건부 렌더링 | 조건에 따라 서로 다른 JSX를 표시하거나 요소를 생략하는 방식입니다. | [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md) | 조건부 클래스 |
