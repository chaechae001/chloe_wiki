# DOM과 이벤트 용어집

이번 학습에서 등장하는 핵심 용어를 코드에서 어떤 역할을 하는지 중심으로 정리했습니다.

## 문서 구조와 선택

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| DOM | HTML 문서를 JavaScript가 읽고 바꿀 수 있는 객체 구조로 표현한 모델 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | document, node |
| document | 현재 웹 문서 전체를 나타내며 DOM 탐색의 시작점이 되는 객체 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | DOM, 선택 메서드 |
| node | 요소, 텍스트처럼 DOM 트리를 구성하는 하나의 객체 단위 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | element, text node |
| element | HTML 태그가 객체로 변환된 요소 노드 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | node |
| DOM 트리 | 포함 관계를 부모·자식·형제 관계로 나타낸 계층 구조 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | parent, child |
| `getElementById()` | 고유한 `id`로 요소 하나를 찾는 메서드 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | 단일 선택 |
| `querySelector()` | CSS 선택자에 맞는 첫 번째 요소를 찾는 메서드 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | CSS 선택자 |
| `querySelectorAll()` | CSS 선택자에 맞는 모든 요소를 정적인 목록으로 반환하는 메서드 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | NodeList |
| HTMLCollection | 태그나 클래스 선택 메서드가 주로 반환하는 요소 모음 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | 반복 처리 |
| NodeList | 노드가 순서대로 담긴 목록 형태의 객체 | [DOM 트리와 요소 선택](01-dom-tree-and-selection.md) | `forEach()` |

## 요소 조작과 생성

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `textContent` | 요소 안의 텍스트를 문자열로 읽거나 바꾸는 프로퍼티 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | `innerHTML` |
| `innerHTML` | 자식 영역을 HTML 문자열로 읽거나 교체하는 프로퍼티 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | XSS, 구조 변경 |
| attribute | `id`, `href`, `placeholder`처럼 태그에 기록된 부가 정보 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | `getAttribute()` |
| `classList` | 요소의 CSS 클래스들을 추가·제거·토글하는 인터페이스 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | 상태 클래스 |
| `createElement()` | 메모리에 새 요소 노드를 만드는 메서드 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | `appendChild()` |
| `appendChild()` | 노드를 부모의 마지막 자식으로 연결하는 메서드 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | DOM 삽입 |
| `insertBefore()` | 기준 자식 노드의 앞에 새 노드를 삽입하는 메서드 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | 형제 순서 |
| `removeChild()` | 부모가 가지고 있는 특정 자식을 제거하는 메서드 | [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md) | 부모·자식 |

## 이벤트

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| event | 클릭, 입력, 키 누름처럼 브라우저에서 발생한 사건 | [이벤트와 핸들러 등록](03-events-and-handlers.md) | 핸들러 |
| 이벤트 핸들러 | 이벤트가 발생했을 때 실행할 함수 | [이벤트와 핸들러 등록](03-events-and-handlers.md) | 콜백 함수 |
| `addEventListener()` | 요소에 이벤트 종류와 핸들러를 연결하는 메서드 | [이벤트와 핸들러 등록](03-events-and-handlers.md) | 함수 참조 |
| 함수 참조 | 함수를 즉시 실행하지 않고 함수 자체를 값으로 전달하는 것 | [이벤트와 핸들러 등록](03-events-and-handlers.md) | `handler` vs `handler()` |
| event 객체 | 발생한 이벤트의 대상·키·좌표·제어 기능을 담은 객체 | [event 객체와 상호작용 패턴](04-event-object-and-interactions.md) | `target` |
| `event.target` | 이벤트가 실제로 시작된 요소 | [event 객체와 상호작용 패턴](04-event-object-and-interactions.md) | 이벤트 위임 |
| `preventDefault()` | 링크 이동이나 폼 제출 같은 브라우저 기본 동작을 취소하는 메서드 | [event 객체와 상호작용 패턴](04-event-object-and-interactions.md) | submit |
| `removeEventListener()` | 이전에 등록한 동일한 함수 참조의 핸들러를 제거하는 메서드 | [event 객체와 상호작용 패턴](04-event-object-and-interactions.md) | 참조 동일성 |

