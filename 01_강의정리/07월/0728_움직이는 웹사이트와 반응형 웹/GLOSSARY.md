# 움직이는 웹사이트와 반응형 웹 용어집

이 학습 묶음에서 사용하는 핵심 용어를 변형, 시간 기반 움직임, 반응형 레이아웃의 학습 흐름에 맞춰 정리했습니다.

## 1. 요소의 모양과 좌표

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `transform` | 요소가 차지한 레이아웃 자리는 유지하면서 화면에 그려지는 모양과 위치를 바꾸는 CSS 속성 | [CSS Transform 기초](01-transform-basics.md) | `transform-origin`, 함수 조합 |
| `rotate()` | 요소를 기준점 주위로 회전시키는 변형 함수로, 양수와 음수 각도로 방향을 구분한다 | [CSS Transform 기초](01-transform-basics.md) | `deg`, `transform-origin` |
| `scale()` | `1`을 원래 크기로 삼아 요소를 비율로 확대하거나 축소하는 변형 함수 | [CSS Transform 기초](01-transform-basics.md) | 배율, 이미지 확대 |
| `skew()` | x축이나 y축을 기준으로 요소를 비스듬하게 기울이는 변형 함수 | [CSS Transform 기초](01-transform-basics.md) | `skewX()`, `skewY()` |
| `translate()` | 요소를 x축과 y축 방향으로 옮겨 보이게 하되 문서 흐름 속 원래 자리는 유지하는 함수 | [CSS Transform 기초](01-transform-basics.md) | 좌표, 음수 이동값 |
| `transform-origin` | 회전과 확대가 시작되는 축을 정하는 속성으로 기본 기준점은 요소의 중앙이다 | [CSS Transform 기초](01-transform-basics.md) | 기준점, 백분율 |
| 함수 조합 | 여러 변형 함수를 하나의 `transform` 값에 이어 쓰는 방식으로, 작성 순서에 따라 결과가 달라진다 | [CSS Transform 기초](01-transform-basics.md) | 좌표계, 선언 순서 |

## 2. 상태 변화와 시간

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `transition` | 한 CSS 상태에서 다른 상태로 값이 바뀔 때 브라우저가 중간 단계를 계산하도록 하는 속성 | [CSS Transition과 Hover](02-transition-and-hover-effects.md) | 시작 상태, 도착 상태 |
| `transition-property` | 어떤 CSS 속성의 변화를 부드럽게 이을지 정하는 항목 | [CSS Transition과 Hover](02-transition-and-hover-effects.md) | `color`, `transform` |
| `duration` | 전환이나 애니메이션 한 회차가 진행되는 시간 | [CSS Transition과 Hover](02-transition-and-hover-effects.md) | `s`, `ms`, `delay` |
| `timing-function` | 전체 시간 안에서 변화 속도가 빨라지고 느려지는 리듬을 정하는 값 | [CSS Transition과 Hover](02-transition-and-hover-effects.md) | `linear`, `ease-out` |
| `delay` | 상태가 바뀌거나 애니메이션이 연결된 뒤 실제 재생을 시작하기 전 기다리는 시간 | [CSS Transition과 Hover](02-transition-and-hover-effects.md) | 시작 시점, `duration` |
| `:hover` | 포인터가 요소 위에 놓인 상태를 선택하는 가상 클래스로, 상호작용의 도착 상태를 만드는 데 자주 쓴다 | [인터랙티브 페이지 애니메이션](04-interactive-page-animation.md) | 기본 상태, 가상 클래스 |
| 상호작용 피드백 | 색·배경·배율의 짧은 변화로 현재 가리키거나 선택한 대상을 사용자에게 알려 주는 반응 | [인터랙티브 페이지 애니메이션](04-interactive-page-animation.md) | 메뉴, 카드, 상태 변화 |

## 3. 키프레임 애니메이션

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `@keyframes` | 애니메이션의 시작·중간·끝 시점에 어떤 스타일을 보여 줄지 정의하는 시간표 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | `from`, `to`, 진행률 |
| `animation-name` | 요소가 사용할 키프레임의 이름을 연결하는 속성 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | 이름 일치, `@keyframes` |
| `animation` | 키프레임 이름, 시간, 반복, 방향 같은 재생 규칙을 묶어 지정하는 속성 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | 단축 속성, 재생 규칙 |
| `iteration-count` | 애니메이션을 몇 회 재생할지 정하며 `infinite`를 쓰면 계속 반복한다 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | 유한 반복, 무한 반복 |
| `animation-direction` | 각 회차를 정방향, 역방향 또는 번갈아 재생할지 정하는 속성 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | `normal`, `alternate` |
| `animation-fill-mode` | 재생 전 대기 시간과 재생 종료 뒤에 어떤 키프레임 상태를 보여 줄지 정하는 속성 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | `forwards`, `backwards`, `both` |
| `alternate` | 홀수 회차와 짝수 회차의 방향을 바꾸어 끝점에서 반대쪽으로 자연스럽게 이어 가는 방향 값 | [CSS 키프레임 애니메이션](03-css-keyframe-animation.md) | 왕복, 반복 횟수 |

## 4. 반응형 화면 조건

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 반응형 웹 | 하나의 콘텐츠 구조를 유지하면서 화면 조건에 따라 크기와 배치 규칙을 바꾸는 설계 방식 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 유동 배치, 콘텐츠 순서 |
| viewport | 브라우저가 웹 콘텐츠를 표시하고 CSS 화면 너비를 판단하는 영역 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | `device-width`, CSS 픽셀 |
| viewport 메타 태그 | 모바일 브라우저의 표시 너비와 초기 배율을 문서에 알려 주는 `<meta>` 설정 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | `width=device-width`, `initial-scale` |
| 미디어쿼리 | 화면 너비 같은 조건이 참일 때만 내부 CSS 규칙을 활성화하는 `@media` 구문 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 조건부 스타일, 캐스케이드 |
| breakpoint | 콘텐츠의 배치나 사용성이 불편해져 다른 규칙으로 전환하기로 정한 경계값 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 경계값 검증, 콘텐츠 기준 |
| `min-width` | viewport가 지정한 너비 이상일 때 참이 되는 미디어 조건 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 최소 경계, 범위 조건 |
| `max-width` | viewport가 지정한 너비 이하일 때 참이 되는 미디어 조건 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 최대 경계, 범위 조건 |
| 캐스케이드 | 적용 가능한 여러 CSS 선언 중 우선순위와 작성 순서 등을 비교해 최종값을 정하는 과정 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) | 기본 규칙 유지, 구체성 |

## 5. 레이아웃 전환과 통합

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| stacking | 넓은 화면에서 나란히 있던 요소를 좁은 화면에서 위아래 한 열로 쌓는 배치 전환 | [반응형 레이아웃 패턴](06-responsive-layout-patterns.md) | `width: 100%`, 문서 순서 |
| `float: none` | 넓은 화면에서 사용한 float 배치를 좁은 화면에서 해제해 요소를 일반 흐름으로 되돌리는 선언 | [반응형 레이아웃 패턴](06-responsive-layout-patterns.md) | 한 열 배치, `clear` |
| `height: auto` | 내용 길이에 맞춰 요소 높이가 늘어나도록 브라우저 계산에 맡기는 값 | [반응형 레이아웃 패턴](06-responsive-layout-patterns.md) | 긴 콘텐츠, 넘침 방지 |
| `max-width: 100%` | 요소의 원래 너비는 유지하되 부모나 화면보다 넓어지지 않도록 제한하는 선언 | [반응형 레이아웃 패턴](06-responsive-layout-patterns.md) | 유동 컨테이너, 이미지 |
| `overflow: hidden` | 자식의 시각적 확대가 컨테이너 경계 밖에 표시되지 않도록 넘친 부분을 자르는 값 | [반응형 모션 워크숍](07-responsive-motion-workshop.md) | 카드 이미지, `scale()` |
| 모션 강도 | 화면 크기와 콘텐츠 목적에 맞춰 이동 거리, 확대 배율, 반복 횟수를 조절한 움직임의 정도 | [반응형 모션 워크숍](07-responsive-motion-workshop.md) | 작은 화면, 유한 반복 |
| 경계값 검증 | breakpoint 바로 앞과 해당 값, 바로 뒤에서 정적 배치와 동적 상태를 비교하는 확인 과정 | [반응형 모션 워크숍](07-responsive-motion-workshop.md) | 넓은 화면, 좁은 화면 |

## 빠른 구분

| 헷갈리는 개념 | 핵심 차이 | 대표 문서 |
|---|---|---|
| `transform` vs 레이아웃 너비 | 그려지는 모양·좌표만 변경 vs 주변 요소가 사용할 공간까지 다시 계산 | [Transform 기초](01-transform-basics.md) |
| `transition` vs `animation` | 상태 변화 사이를 연결 vs 키프레임 시간표를 독립적으로 재생 | [반응형 모션 워크숍](07-responsive-motion-workshop.md) |
| 미디어쿼리 vs `:hover` | 화면 환경 조건을 판단 vs 요소의 포인터 상태를 판단 | [반응형 모션 워크숍](07-responsive-motion-workshop.md) |
| `width: 100%` vs `max-width: 100%` | 현재 컨테이너 폭을 사용 vs 원래 크기를 넘지 않으면서 상한만 제한 | [반응형 레이아웃 패턴](06-responsive-layout-patterns.md) |
| CSS 상속 vs 기본 선언 유지 | 부모의 계산값을 전달받음 vs 같은 요소의 기존 규칙이 캐스케이드에 남음 | [반응형 웹과 viewport](05-responsive-web-and-viewport.md) |
