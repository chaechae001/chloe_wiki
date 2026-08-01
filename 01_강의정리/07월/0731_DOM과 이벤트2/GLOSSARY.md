# DOM과 이벤트 2 용어집

이벤트 전파와 상호작용 구현에서 자주 만나는 용어를 역할 중심으로 정리했습니다.

## 이벤트 전파

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 이벤트 전파 | 하나의 이벤트가 DOM 경로를 따라 전달되는 현상 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | 이벤트 경로 |
| 이벤트 경로 | 이벤트 대상과 그 조상 요소들로 이어지는 이동 경로 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | DOM 트리 |
| 캡처링 | 상위 객체에서 이벤트 대상 방향으로 내려오는 단계 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | capture 옵션 |
| 타깃 단계 | 이벤트가 실제 발생 대상에 도착한 단계 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | `target` |
| 버블링 | 이벤트 대상에서 상위 요소 방향으로 올라가는 단계 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | 이벤트 위임 |
| `eventPhase` | 현재 전파 단계가 캡처링·타깃·버블링 중 무엇인지 나타내는 값 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | 1, 2, 3 |
| `stopPropagation()` | 현재 이벤트가 전파 경로의 다음 요소로 이동하는 것을 막는 메서드 | [이벤트 전파의 세 단계](01-event-propagation-phases.md) | 전파 제어 |

## 대상과 위임

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `event.target` | 이벤트가 실제로 시작된 가장 안쪽 요소 | [이벤트 대상과 위임](02-event-targets-and-delegation.md) | 발생 지점 |
| `event.currentTarget` | 현재 실행 중인 핸들러가 등록된 요소 | [이벤트 대상과 위임](02-event-targets-and-delegation.md) | 리스너 위치 |
| 이벤트 위임 | 부모의 리스너 하나로 여러 자식의 이벤트를 처리하는 패턴 | [이벤트 대상과 위임](02-event-targets-and-delegation.md) | 버블링 |
| `closest()` | 자신부터 조상 방향으로 조건에 맞는 가장 가까운 요소를 찾는 메서드 | [이벤트 대상과 위임](02-event-targets-and-delegation.md) | CSS 선택자 |
| `contains()` | 특정 노드가 요소 내부에 포함되는지 확인하는 메서드 | [이벤트 대상과 위임](02-event-targets-and-delegation.md) | 범위 검증 |

## 폼과 UI 상태

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `submit` 이벤트 | 버튼 클릭이나 Enter 입력 등 폼 제출 시도를 대표하는 이벤트 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | form |
| `preventDefault()` | 폼 제출이나 링크 이동 같은 브라우저 기본 동작을 취소하는 메서드 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | 검증 |
| `input` 이벤트 | 입력 요소의 값이 실제로 바뀔 때 발생하는 이벤트 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | `keyup` |
| `disabled` | 사용자가 컨트롤을 조작하거나 제출하지 못하게 하는 상태 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | 버튼 상태 |
| UI 상태 | 입력 유효성·선택·진행 중처럼 화면이 기억하고 표현해야 하는 현재 조건 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | 클래스 |
| `aria-live` | 동적으로 바뀐 메시지를 보조기술에 알리는 접근성 속성 | [폼과 UI 상태 관리](03-forms-and-ui-state.md) | 상태 메시지 |

## 동적 UI와 시간 제어

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 동적 목록 | 사용자 입력이나 데이터에 따라 항목이 실행 중 생성·삭제되는 목록 | [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md) | 이벤트 위임 |
| 필터링 | 검색 조건과 맞지 않는 항목을 숨기고 일치 항목만 보여주는 처리 | [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md) | 문자열 정규화 |
| `setInterval()` | 지정한 시간 간격마다 함수를 반복 호출하는 타이머 | [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md) | `clearInterval()` |
| 타이머 ID | 실행 중인 타이머를 나중에 중단하기 위한 식별값 | [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md) | 수명주기 |
| 경쟁 상태 | 여러 비동기 흐름이 거의 동시에 조건을 만족해 결과가 중복 결정될 수 있는 상황 | [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md) | 종료 플래그 |

