# 용어집

JSX, 컴포넌트, Props와 State에서 반복되는 핵심 용어를 정리했습니다.

## JSX와 렌더링

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| JSX | JavaScript 안에서 UI 구조를 HTML과 유사하게 표현하는 문법입니다. | [JSX 문법과 기본 규칙](01-jsx-syntax-and-rules.md) | Babel, 표현식 |
| Fragment | 실제 DOM 요소를 추가하지 않고 여러 형제 JSX를 묶는 요소입니다. | [JSX 문법과 기본 규칙](01-jsx-syntax-and-rules.md) | 최상위 요소 |
| className | JSX에서 CSS 클래스 이름을 지정하는 속성입니다. | [스타일과 React DOM 속성](02-styles-and-dom-attributes.md) | style 객체 |
| 렌더 단계 | Props와 State로 다음 UI 결과를 계산하는 단계입니다. | [컴포넌트와 children](03-components-children-and-rendering.md) | 커밋 단계 |
| 커밋 단계 | 계산 결과에 따라 실제 DOM 변경을 반영하는 단계입니다. | [컴포넌트와 children](03-components-children-and-rendering.md) | 렌더링 |

## 컴포넌트와 데이터

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 함수 컴포넌트 | Props를 받아 렌더 가능한 JSX를 반환하는 JavaScript 함수입니다. | [컴포넌트와 children](03-components-children-and-rendering.md) | 재사용 |
| children | 컴포넌트의 여는 태그와 닫는 태그 사이에 전달된 내용입니다. | [컴포넌트와 children](03-components-children-and-rendering.md) | 조합 |
| Props | 부모가 자식에게 전달하는 읽기 전용 입력입니다. | [Props와 단방향 데이터 흐름](04-props-and-data-flow.md) | 콜백 Props |
| 단방향 데이터 흐름 | 데이터가 부모에서 자식 방향으로 전달되는 구조입니다. | [Props와 단방향 데이터 흐름](04-props-and-data-flow.md) | 상태 끌어올리기 |
| State | 컴포넌트가 렌더링 사이에 기억하고 setter로 갱신하는 데이터입니다. | [State와 불변 갱신](05-state-and-immutable-updates.md) | useState |
| 함수형 갱신 | 최신 이전 State를 받아 다음 State를 계산하는 setter 사용법입니다. | [State와 불변 갱신](05-state-and-immutable-updates.md) | updater 함수 |
| 불변성 | 기존 객체를 직접 바꾸지 않고 변경된 새 객체를 만드는 원칙입니다. | [State와 불변 갱신](05-state-and-immutable-updates.md) | 참조 동일성 |

## 목록과 입력

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| key | 형제 목록에서 React가 항목의 정체성을 비교하도록 돕는 값입니다. | [목록 key와 입력 상태 설계](06-list-keys-and-form-state.md) | 고유 id |
| 제어 컴포넌트 | 입력의 현재값을 React State가 관리하는 입력 요소입니다. | [목록 key와 입력 상태 설계](06-list-keys-and-form-state.md) | value, onChange |
| 비제어 컴포넌트 | 현재 입력값을 DOM이 관리하고 필요할 때 참조로 읽는 입력 방식입니다. | [스타일과 React DOM 속성](02-styles-and-dom-attributes.md) | defaultValue |

## 빠른 비교

| 비교 | 핵심 차이 |
|---|---|
| Props vs State | Props는 외부 입력, State는 내부에서 기억하고 갱신하는 값입니다. |
| 렌더 vs 커밋 | 렌더는 UI 계산, 커밋은 실제 DOM 반영입니다. |
| value vs defaultValue | value는 현재값 제어, defaultValue는 초기값 제공입니다. |
| key vs id | key는 React 비교용, id는 애플리케이션 데이터 식별용입니다. |
