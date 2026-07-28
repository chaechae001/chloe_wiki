# 반응형 모션 워크숍 — 크기 변화와 사용자 반응을 한 화면에 연결하기

> 움직임이 자연스러워도 작은 화면에서 레이아웃이 깨지면 좋은 인터페이스가 아닙니다. 반대로 배치가 안정적이어도 사용자의 행동에 아무 반응이 없으면 클릭 가능한 요소를 알아보기 어렵습니다.

`transform` · `transition` · `animation` · `media-query` · `interaction`

## 핵심요약
- 통합 화면은 기본 레이아웃, 반응형 재배치, 상호작용 효과의 순서로 구현하면 디버깅하기 쉽다.
- `transform`은 요소의 시각적 위치·크기·각도를 바꾸고, `transition`은 상태 변화 사이를 이어 준다.
- 반복되는 장식 동작은 `@keyframes`와 `animation`으로 정의하되 콘텐츠 읽기를 방해하지 않아야 한다.
- 미디어쿼리에서는 레이아웃뿐 아니라 작은 화면에 맞는 이동 거리와 효과 강도도 조정할 수 있다.
- 결과 검증은 넓은 화면, 경계값, 좁은 화면에서 정적 배치와 동적 상태를 모두 확인해야 한다.
## 1. 통합 순서: 구조가 먼저, 움직임은 다음
### 1) 정의
반응형 모션 화면은 화면 크기에 따른 배치 변화와 사용자 행동에 따른 시각적 변화를 한 컴포넌트 안에서 함께 다루는 구현입니다.
### 2) 왜 필요한가
레이아웃과 모션을 동시에 무작정 작성하면 카드가 이동한 것인지, 열이 바뀐 것인지 원인을 구분하기 어렵습니다.
구현 순서를 나누면 각 단계의 책임이 명확해집니다.
### 3) 핵심 흐름 재구성
다음 순서가 안정적입니다.
1. 의미 순서에 맞는 HTML을 작성한다.
2. 넓은 화면의 기본 너비와 배치를 만든다.
3. 미디어쿼리로 좁은 화면의 열과 정렬을 바꾼다.
4. `:hover` 같은 상태에 최종 스타일을 선언한다.
5. `transition`으로 두 상태 사이의 변화를 연결한다.
6. 반복 동작이 필요할 때만 `@keyframes`를 추가한다.
이 순서에서는 움직임을 잠시 제거해도 페이지의 정보 구조가 그대로 유지됩니다.
### 4) 쉬운 예시
전시장을 꾸밀 때 작품을 어디에 놓을지 먼저 정하고, 그다음 조명 전환과 안내 표시의 움직임을 추가하는 것과 같습니다.
작품 배치가 안정되어야 조명 효과가 어느 위치에서 어떻게 보일지 판단할 수 있습니다.
### 5) 코드 예시
```css
.gallery-card {
  float: left;
  width: 50%;
}
@media (max-width: 800px) {
  .gallery-card {
    float: none;
    width: 100%;
  }
}
.gallery-card img {
  transition: transform 0.3s ease;
}
.gallery-card:hover img {
  transform: scale(1.05);
}
```
### 6) 헷갈리는 점
미디어쿼리와 `:hover`는 서로 대체하는 기능이 아닙니다.
미디어쿼리는 환경 조건을, `:hover`는 요소의 상호작용 상태를 판단하며 두 조건이 동시에 적용될 수도 있습니다.
### 7) 한 줄 정리
> 반응형 모션은 배치를 먼저 안정화한 뒤 상태 변화와 시간 흐름을 얹을 때 이해하고 고치기 쉽다.

## 2. transform과 transition을 반응형 카드에 적용하기
### 1) 정의
`transform`은 요소를 이동·회전·확대·축소하는 속성이고, `transition`은 한 CSS 상태에서 다른 상태로 바뀌는 시간을 설정합니다.
### 2) 왜 필요한가
카드 이미지가 살짝 커지거나 링크 색이 부드럽게 바뀌면 사용자는 현재 가리키는 대상을 빠르게 알아차릴 수 있습니다.
하지만 확대 폭이 너무 크면 옆 카드나 화면 가장자리를 침범할 수 있으므로 컨테이너와 화면 크기를 함께 고려해야 합니다.
### 3) 핵심 흐름 재구성
상태 변화는 시작값과 끝값으로 나눠 읽습니다.
| 단계 | 이미지 배율 | 카드 배경 | 의미 |
|---|---:|---|---|
| 기본 | `1` | 연한 파랑 | 사용 전 상태 |
| hover | `1.06` | 진한 파랑 | 현재 가리키는 상태 |
| 좁은 화면 hover | `1.03` | 진한 파랑 | 작은 화면에서 효과 축소 |
`transition`은 기본 상태에 두는 것이 좋습니다.
그러면 hover에 들어갈 때와 빠져나올 때 모두 같은 시간 규칙으로 연결됩니다.
### 4) 쉬운 예시
서랍 손잡이에 손을 대면 작은 표시등이 서서히 밝아지는 장면을 떠올릴 수 있습니다.
표시등은 손잡이 위치를 바꾸지 않으면서도 현재 선택된 대상을 알려 줍니다.
### 5) 코드 예시
```css
.gallery-card {
  overflow: hidden;
  background: #e7f5ff;
  transition: background-color 0.3s ease;
}
.gallery-card img {
  display: block;
  width: 100%;
  transition: transform 0.3s ease;
}
.gallery-card:hover {
  background: #4dabf7;
}
.gallery-card:hover img {
  transform: scale(1.06);
}
@media (max-width: 800px) {
  .gallery-card:hover img {
    transform: scale(1.03);
  }
}
```
### 6) 헷갈리는 점
`transform: scale()`은 주변 요소가 차지하는 레이아웃 공간을 다시 계산하지 않습니다.
이미지가 시각적으로 커지므로 카드 바깥으로 보일 수 있고, 이 예제에서는 부모의 `overflow: hidden`으로 확대 영역을 카드 안에 제한합니다.
### 7) 한 줄 정리
> 카드 모션은 끝 상태만 만드는 것이 아니라 컨테이너 범위와 전환 시간을 함께 설계해야 자연스럽다.

## 3. animation과 화면 크기별 강도 조정
### 1) 정의
CSS animation은 `@keyframes`에 여러 시점의 스타일을 정의하고 `animation` 속성으로 반복 시간과 횟수 등을 연결하는 기능입니다.
### 2) 왜 필요한가
사용자 행동을 기다리지 않고도 새 알림이나 주목할 제목처럼 변화가 있는 요소를 짧게 강조할 수 있습니다.
반복 효과는 시선을 강하게 끌기 때문에 꼭 필요한 대상과 횟수에만 적용해야 합니다.
### 3) 핵심 흐름 재구성
다음 예제는 배지가 위아래로 움직이는 세 시점을 사용합니다.
```css
@keyframes badge-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
```
시작과 끝을 같은 값으로 두면 반복 경계에서 갑자기 위치가 튀는 느낌을 줄일 수 있습니다.
좁은 화면에서는 이동 거리를 `-4px`로 줄인 별도 키프레임을 연결할 수 있습니다.
### 4) 쉬운 예시
매장 안내판의 작은 화살표가 잠깐 위아래로 움직여 새 소식을 가리키는 장면과 비슷합니다.
화살표가 계속 크게 움직이면 글 읽기를 방해하지만, 짧고 작은 동작은 위치를 알리는 데 도움이 됩니다.
### 5) 코드 예시
```css
.new-badge {
  display: inline-block;
  animation: badge-bounce 0.8s ease-in-out 3;
}
@keyframes badge-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
@keyframes badge-bounce-small {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-4px); }
}
@media (max-width: 800px) {
  .new-badge {
    animation-name: badge-bounce-small;
  }
}
```
### 6) 헷갈리는 점
`transition`은 상태가 달라지는 계기가 있어야 시작하지만, `animation`은 적용되는 순간 자체적으로 실행할 수 있습니다.
단순 hover 변화에는 transition이 간결하고, 여러 중간 단계나 정해진 반복에는 animation이 알맞습니다.
### 7) 한 줄 정리
> animation은 독립된 시간표를 가진 움직임이며 화면이 좁을수록 거리와 반복 강도도 함께 점검한다.

## 코드로 보기 — 반응형 갤러리와 상태 피드백
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<header class="workshop-header">
  <h1>주말 전시 <span class="new-badge">NEW</span></h1>
  <nav><a href="#gallery">작품</a><a href="#guide">안내</a></nav>
</header>
<main id="gallery" class="gallery">
  <article class="gallery-card"><div class="picture blue"></div><h2>바다</h2></article>
  <article class="gallery-card"><div class="picture pink"></div><h2>노을</h2></article>
  <article class="gallery-card"><div class="picture green"></div><h2>숲</h2></article>
  <article class="gallery-card"><div class="picture yellow"></div><h2>햇빛</h2></article>
</main>
<footer id="guide" class="workshop-footer">
  <p>작품은 화면 크기에 맞춰 배치됩니다.</p>
  <p>카드 위에서 색과 크기 변화를 확인하세요.</p>
</footer>
```
```css
* { box-sizing: border-box; }
body { width: 960px; max-width: 100%; margin: 0 auto; color: #212529; }
.workshop-header, .gallery, .workshop-footer { display: flow-root; }
.workshop-header h1, .workshop-header nav { float: left; width: 50%; padding: 24px; }
.workshop-header nav a { display: inline-block; width: 50%; text-align: center; color: inherit; transition: color .3s; }
.workshop-header nav a:hover { color: #7048e8; }
.gallery-card { float: left; width: 50%; overflow: hidden; transition: background-color .3s; }
.gallery-card .picture { height: 220px; transition: transform .3s; }
.gallery-card:hover { background: #f1f3f5; }
.gallery-card:hover .picture { transform: scale(1.06); }
.gallery-card h2 { margin: 0; padding: 16px; text-align: center; }
.blue { background: #74c0fc; } .pink { background: #faa2c1; }
.green { background: #8ce99a; } .yellow { background: #ffe066; }
.new-badge { display: inline-block; font-size: .6em; animation: badge-bounce .8s ease-in-out 3; }
.workshop-footer p { float: left; width: 50%; margin: 0; padding: 20px; }
@keyframes badge-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
@media (max-width: 800px) {
  .workshop-header h1, .workshop-header nav,
  .gallery-card, .workshop-footer p { float: none; width: 100%; }
  .workshop-header h1, .workshop-footer p { text-align: center; }
  .gallery-card .picture { height: 280px; }
  .gallery-card:hover .picture { transform: scale(1.03); }
}
```
### 코드 목적
상단·본문·하단의 반응형 배치에 링크 transition, 카드 transform, 배지 animation을 결합합니다.
### 코드 흐름
1. viewport를 실제 기기 너비에 맞춘다.
2. 넓은 화면에서 상단, 카드, 하단을 `50%` 열로 배치한다.
3. `800px` 이하에서는 각 열을 `100%` 세로 흐름으로 바꾼다.
4. 링크 색과 카드 확대를 `0.3s` transition으로 연결한다.
5. 배지는 `0.8s` 동안 세 번만 위아래로 움직인다.
### 예상 렌더링
```text
960px: 상단 2열 / 카드 2열 / 하단 2열
800px 이하: 모든 주요 구역 1열
카드 기본 배율 1 → hover 배율 1.06(넓은 화면), 1.03(좁은 화면)
배지 이동 0px → -8px → 0px, 총 3회
링크 색 전환 시간 0.3초
```
### 실행 결과 해석
`800px` 이하에서 네 카드는 각각 전체 너비를 차지해 위에서 아래로 쌓입니다.
같은 구간에서 hover 배율은 `1.06`에서 `1.03`으로 줄어 작은 화면 가장자리에서 시각적 넘침이 덜합니다.
배지는 무한 반복하지 않고 세 번 뒤 멈추므로 처음에는 주목을 끌지만 이후 콘텐츠 읽기를 계속 방해하지 않습니다.
### 실무 연결
포트폴리오 프로젝트 목록, 제품 카드, 콘텐츠 갤러리처럼 화면별 열 수가 달라지고 선택 상태를 시각적으로 알려야 하는 페이지에 적용할 수 있습니다.
터치 환경에는 hover가 항상 같은 방식으로 나타나지 않으므로, 중요한 정보는 모션에만 의존하지 않고 제목과 링크 텍스트로도 전달해야 합니다.
## 직접 해보기
1. 카드 확대 효과를 부드럽게 만들려면 시작 상태의 어느 요소에 어떤 속성을 선언해야 하나요?
2. `800px` 이하에서 카드 네 개를 한 열로 만들고 확대 폭을 절반가량 줄이는 CSS를 작성해 보세요.
3. 알림 배지가 계속 움직여 글 읽기를 방해한다면 반복 횟수와 이동 거리를 어떻게 조정할 수 있나요?
<details>
<summary>정답 보기</summary>

1. 변하는 대상인 카드 이미지에 `transition: transform 0.3s ease`를 선언합니다. 기본 상태에 두어 들어갈 때와 나올 때 모두 전환되게 합니다.
2. 미디어쿼리에서 카드에 `width: 100%`와 필요한 float 해제를 적용하고, hover 이미지에는 `transform: scale(1.03)`처럼 작은 배율을 선언합니다.
3. `infinite` 대신 `3`처럼 유한한 횟수를 쓰고 `translateY(-8px)`을 `-4px`처럼 줄일 수 있습니다. 강조 목적을 달성하는 최소 움직임을 선택합니다.

</details>
## 헷갈리기 쉬운 포인트
| 헷갈리는 개념 | 차이 |
|---|---|
| `transform` vs 레이아웃 너비 변경 | 시각적 모양과 좌표를 바꿈 vs 다른 요소가 차지할 공간까지 다시 계산 |
| `transition` vs `animation` | 상태 변화 사이를 연결 vs 키프레임 시간표를 자체 실행 |
| 미디어쿼리 vs hover | 화면 환경 조건을 판단 vs 포인터가 요소 위에 있는 상태를 판단 |
| 무한 반복 vs 유한 반복 | 계속 움직여 주의를 지속적으로 끎 vs 정해진 횟수 뒤 멈춤 |
## 연결되는 개념
- 이전에 알면 좋은 개념: [반응형 레이아웃 패턴](06-responsive-layout-patterns.md)
- 다음에 이어지는 개념: [전체 학습 흐름](OVERVIEW.md)
- 함께 보면 좋은 키워드: `state-change`, `keyframes`, `breakpoint`
## 셀프 체크
- [ ] 반응형 배치와 모션을 구현하는 순서를 설명할 수 있다.
- [ ] transform이 레이아웃 공간에 미치는 영향을 말할 수 있다.
- [ ] transition과 animation의 사용 상황을 구분할 수 있다.
- [ ] 작은 화면에서 모션 강도를 조정할 수 있다.
- [ ] 세 화면 구간에서 정적·동적 상태를 함께 검사할 수 있다.
### 복습 질문 및 답변
**Q1. 카드 너비를 바꾸기 전에 모션부터 구현하면 왜 디버깅이 어려운가요?**
<details>
<summary>답</summary>

카드가 예상 위치를 벗어났을 때 원인이 열 전환인지 transform인지 분리하기 어렵기 때문입니다. 기본 배치와 반응형 배치를 먼저 확인한 뒤 모션을 추가하면 원인의 범위가 줄어듭니다.

</details>
**Q2. 이미지가 확대될 때 옆 카드 위로 보이는 현상을 어떻게 제한할 수 있나요?**
<details>
<summary>답</summary>

이미지를 감싼 카드에 `overflow: hidden`을 적용해 시각적 확대가 카드 경계를 넘어 표시되지 않게 할 수 있습니다. 확대 배율 자체도 화면에 맞게 줄여야 합니다.

</details>
**Q3. 완성한 화면을 어떤 순서로 검증하면 좋을까요?**
<details>
<summary>답</summary>

먼저 넓은 화면에서 각 구역의 열과 기본 상태를 확인합니다. 다음으로 breakpoint 바로 위와 아래에서 재배치를 비교하고, 좁은 화면에서 긴 텍스트와 카드 넘침을 봅니다. 마지막으로 각 구간에서 hover 전환과 반복 animation의 거리·시간·횟수를 확인합니다.

</details>
## 한 줄 정리
> 반응형 모션 화면은 정보 구조와 열 전환을 먼저 안정화하고, transform·transition·animation을 화면 크기에 맞는 강도로 연결할 때 완성된다.
