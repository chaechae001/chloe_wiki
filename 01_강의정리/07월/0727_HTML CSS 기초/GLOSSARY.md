# 용어집

이번 회차에서 다룬 HTML 구조와 CSS 레이아웃 용어를 쉬운 말로 정리했습니다. 정의만 외우기보다 각 용어가 정보의 의미, 스타일의 결정, 공간의 계산, 화면의 흐름 중 어디에 관여하는지 함께 살펴보세요.

## 빠른 탐색

- 문서의 의미와 품질 기준이 궁금하다면 **웹과 HTML 구조**부터 확인합니다.
- 스타일이 예상과 다르게 적용된다면 **CSS 선택과 우선순위**를 확인합니다.
- 요소의 너비나 여백이 맞지 않는다면 **박스와 크기 계산**을 확인합니다.
- 요소가 줄을 바꾸거나 떠서 배치된다면 **흐름과 레이아웃**을 확인합니다.

## 웹과 HTML 구조

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| HTML | 콘텐츠가 제목인지 문단인지 탐색 영역인지처럼 정보의 의미와 문서 구조를 표현하는 언어 | [1편](01-web-foundations-and-standards.md) | 시맨틱 HTML, 문서 골격 |
| CSS | HTML 요소의 색상, 크기, 간격, 배치처럼 시각적 표현을 정하는 스타일 언어 | [1편](01-web-foundations-and-standards.md) | 선택자, 박스 모델 |
| JavaScript | 사용자 입력에 반응하거나 화면 상태를 바꾸는 동작을 담당하는 프로그래밍 언어 | [1편](01-web-foundations-and-standards.md) | HTML, CSS |
| 웹 표준 | 서로 다른 도구와 환경에서도 웹 문서를 일관되게 해석할 수 있도록 합의한 공통 규칙 | [1편](01-web-foundations-and-standards.md) | 크로스 브라우징, 접근성 |
| 웹 접근성 | 장애나 기기, 입력 방식의 차이와 관계없이 누구나 정보와 기능에 도달할 수 있게 설계하는 원칙 | [1편](01-web-foundations-and-standards.md) | 시맨틱 HTML, 대체 텍스트 |
| 크로스 브라우징 | 브라우저가 달라도 핵심 정보와 기능을 이용할 수 있도록 호환성을 확인하고 보완하는 작업 | [1편](01-web-foundations-and-standards.md) | 웹 표준, 점진적 개선 |
| 시맨틱 HTML | 화면 모양이 아니라 콘텐츠의 역할이 드러나는 태그를 골라 문서 구조를 만드는 방식 | [2편](02-semantic-html-structure.md) | `header`, `nav`, `main`, `article` |
| 문서 골격 | 선언과 `html`, `head`, `body`가 문서 정보와 화면 콘텐츠를 나누어 감싸는 기본 구조 | [2편](02-semantic-html-structure.md) | 문자 인코딩, `header` |
| 제목 계층 | `h1`부터 `h6`까지 내용의 상하 관계를 단계적으로 나타내는 구조로, 글자 크기만을 위한 장식이 아님 | [2편](02-semantic-html-structure.md) | 문서 개요, 접근성 |

## CSS 선택과 우선순위

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| CSS 규칙 | `선택자 { 속성: 값; }` 형태로 어떤 요소의 무엇을 어떻게 보일지 선언한 한 묶음 | [3편](03-css-selectors-and-cascade.md) | 선택자, 선언 |
| 타입 선택자 | `p`, `li`처럼 같은 태그 이름을 가진 모든 요소를 넓게 고르는 선택자 | [3편](03-css-selectors-and-cascade.md) | 클래스 선택자, 기본 스타일 |
| 클래스 선택자 | `.card`처럼 여러 요소에 재사용할 수 있는 이름으로 대상을 고르는 선택자 | [3편](03-css-selectors-and-cascade.md) | ID 선택자, 구체성 |
| ID 선택자 | `#hero`처럼 문서에서 고유한 식별자를 가진 요소를 고르는 선택자 | [3편](03-css-selectors-and-cascade.md) | 클래스 선택자, 구체성 |
| 후손 선택자 | `.menu a`처럼 특정 요소 안쪽에 있는 후손만 골라 스타일 범위를 좁히는 선택자 | [3편](03-css-selectors-and-cascade.md) | 선택 범위, 구체성 |
| 상속 | 부모에 지정한 일부 글자 관련 속성을 자식이 이어받는 현상으로, 모든 CSS 속성에 적용되지는 않음 | [3편](03-css-selectors-and-cascade.md) | 캐스케이드, 직접 선언 |
| 구체성 | 여러 선택자가 같은 요소를 가리킬 때 어떤 선택자가 더 구체적인지 비교하는 우선순위 점수 | [3편](03-css-selectors-and-cascade.md) | ID, 클래스, 타입 선택자 |
| 캐스케이드 | 여러 선언이 충돌할 때 중요도, 선언 위치, 구체성, 작성 순서를 따져 최종 스타일을 정하는 과정 | [3편](03-css-selectors-and-cascade.md) | 상속, 구체성 |

## 박스와 크기 계산

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 박스 모델 | 모든 요소의 공간을 안쪽부터 내용, 패딩, 테두리, 마진의 네 겹으로 설명하는 규칙 | [4편](04-css-box-model.md) | `box-sizing`, 실제 너비 |
| content | 글이나 이미지가 놓이는 박스의 가장 안쪽 내용 영역 | [4편](04-css-box-model.md) | `content-box`, 패딩 |
| padding | 내용과 테두리 사이의 안쪽 여백으로, 요소의 배경이 함께 칠해지는 영역 | [4편](04-css-box-model.md) | content, border |
| border | 패딩 바깥을 둘러싸 요소의 경계를 만드는 선으로, 보이는 박스 크기에 포함됨 | [4편](04-css-box-model.md) | padding, margin |
| margin | 테두리 바깥에서 이웃 요소와 거리를 만드는 투명한 바깥 여백 | [4편](04-css-box-model.md) | 마진 병합, padding |
| `content-box` | 선언한 `width`와 `height`를 내용 영역의 크기로 보고 패딩과 테두리를 바깥에 더하는 기본 계산 방식 | [4편](04-css-box-model.md) | 실제 너비, `border-box` |
| `border-box` | 선언한 너비와 높이 안에 패딩과 테두리까지 포함해 보이는 크기를 계산하는 방식 | [4편](04-css-box-model.md) | `box-sizing`, 열 너비 계산 |

## 흐름과 레이아웃

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 일반 흐름 | 별도 배치 속성이 없을 때 요소가 HTML 순서와 표시 방식에 따라 자연스럽게 자리를 잡는 기본 규칙 | [6편](06-float-clear-and-layout-flow.md) | block, inline, float |
| block | 새 줄에서 시작하고 기본적으로 가로 공간을 넓게 차지해 문서의 구역을 만드는 표시 방식 | [5편](05-display-and-margin-collapse.md) | inline, `display` |
| inline | 글의 흐름 안에서 내용만큼 자리를 차지하며 이어지는 표시 방식 | [5편](05-display-and-margin-collapse.md) | block, `inline-block` |
| `inline-block` | 한 줄에 나란히 놓이면서 너비와 높이도 지정할 수 있는 혼합 표시 방식 | [5편](05-display-and-margin-collapse.md) | inline, `display` |
| `display` | 요소가 블록, 인라인, 인라인 블록 등 어떤 방식으로 레이아웃에 참여할지 정하는 속성 | [5편](05-display-and-margin-collapse.md) | 요소의 의미, 일반 흐름 |
| 마진 병합 | 일반 흐름에서 맞닿은 블록의 세로 마진이 단순히 더해지지 않고 하나의 간격으로 합쳐지는 현상 | [5편](05-display-and-margin-collapse.md) | margin, 블록 서식 맥락 |
| `float` | 상자를 왼쪽이나 오른쪽에 붙이고 뒤따르는 인라인 콘텐츠가 그 둘레를 흐르게 하는 속성 | [6편](06-float-clear-and-layout-flow.md) | 일반 흐름, 부모 높이 |
| `clear` | 앞선 float 옆에 다음 요소가 놓이지 않도록 자기 시작 위치를 필요한 만큼 아래로 옮기는 속성 | [6편](06-float-clear-and-layout-flow.md) | float, 흐름 복구 |

## 용어를 연결해 보는 순서

1. 시맨틱 HTML로 정보의 역할을 정한 뒤 CSS 선택자로 표현 대상을 고릅니다.
2. 캐스케이드로 최종 스타일을 판단하고 박스 모델로 요소가 차지할 공간을 계산합니다.
3. `display`로 기본 자리 잡기를 확인한 뒤 마진 병합과 float가 흐름을 어떻게 바꾸는지 추적합니다.
4. 통합 예시는 [페이지 레이아웃 워크숍](07-page-layout-workshop.md)에서 확인할 수 있습니다.
