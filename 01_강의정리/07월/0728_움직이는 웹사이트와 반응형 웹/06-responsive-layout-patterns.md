# 반응형 레이아웃 패턴 — header·main·footer를 다시 흐르게 만들기

> 반응형 페이지는 모든 값을 작게 줄인 화면이 아닙니다. 상단, 본문, 하단이 좁은 공간에서도 읽는 순서와 역할을 유지하도록 열을 행으로 바꾸는 작업이 핵심입니다.

`header` · `main` · `footer` · `stacking` · `responsive-layout`

## 핵심요약
- 데스크톱의 가로 배치는 모바일에서 위아래 흐름으로 전환하는 경우가 많다.
- header는 로고와 내비게이션의 너비, 높이, 정렬을 함께 바꿔야 겹침을 막을 수 있다.
- main의 `50%` 카드를 `100%`로 바꾸면 두 열이 한 열로 재배치된다.
- footer는 정보 블록을 세로로 쌓고 고정 높이 대신 콘텐츠가 늘어날 여지를 둬야 한다.
- 반응형 검사는 특정 화면 한 장보다 경계값 앞뒤와 긴 콘텐츠 상황까지 포함해야 한다.
## 1. header: 로고와 메뉴의 우선순위
### 1) 정의
header 반응형 패턴은 넓은 화면에서 나란히 놓인 브랜드 영역과 내비게이션을 좁은 화면에서 두 줄로 재배치하는 방식입니다.
### 2) 왜 필요한가
로고와 메뉴가 각각 절반 너비를 사용하는 데스크톱 구조는 공간이 줄면 메뉴 글자가 겹치거나 줄바꿈될 수 있습니다.
모바일에서는 로고를 첫 줄 전체에, 메뉴를 둘째 줄 전체에 두면 정보 계층과 클릭 영역을 모두 유지하기 쉽습니다.
### 3) 핵심 흐름 재구성
넓은 화면의 구조를 먼저 읽습니다.
```text
┌──────────────────────────────────────────┐
│ 로고 50%       │ 메뉴 1 │ 메뉴 2 │ 메뉴 3 │
└──────────────────────────────────────────┘
```
좁은 화면에서는 다음처럼 바꿉니다.
```text
┌──────────────────────────────────────────┐
│                 로고 100%                │
├──────────────┬──────────────┬────────────┤
│    메뉴 1    │    메뉴 2    │    메뉴 3   │
└──────────────┴──────────────┴────────────┘
```
이때 너비만 바꾸면 header의 기존 높이가 부족할 수 있습니다.
한 줄이 두 줄로 늘었으므로 상단 전체 높이 또는 자동 높이도 함께 확인합니다.
### 4) 쉬운 예시
큰 안내판에서는 상호와 메뉴를 같은 줄에 놓을 수 있지만, 좁은 입간판에서는 상호를 위에 크게 쓰고 메뉴를 아래에 나누는 편이 읽기 쉽습니다.
### 5) 코드 예시
```html
<header class="site-header">
  <h1 class="brand"><a href="#">작은 서재</a></h1>
  <nav class="menu" aria-label="주요 메뉴">
    <a href="#books">책</a>
    <a href="#notes">기록</a>
    <a href="#about">소개</a>
  </nav>
</header>
```
```css
.brand,
.menu {
  float: left;
  width: 50%;
  height: 80px;
}
@media (max-width: 800px) {
  .brand,
  .menu {
    width: 100%;
  }
  .site-header {
    min-height: 160px;
  }
}
```
### 6) 헷갈리는 점
자식 두 개를 `width: 100%`로 바꾸면 한 줄에 200%를 차지하는 것이 아닙니다.
각 요소가 한 줄의 전체 너비를 차지하면서 다음 요소가 아래 줄로 이동합니다.
### 7) 한 줄 정리
> 모바일 header는 로고와 메뉴의 너비뿐 아니라 두 줄이 된 뒤의 높이와 정렬까지 함께 설계한다.

## 2. main: 두 열 카드를 한 열로 전환하기
### 1) 정의
본문의 열 전환은 여러 콘텐츠 카드를 넓은 화면에서는 나란히, 좁은 화면에서는 세로 목록으로 배치하는 패턴입니다.
### 2) 왜 필요한가
두 열의 카드가 좁은 화면에서도 `50%`를 유지하면 이미지와 제목이 너무 작아지고 클릭 영역도 답답해집니다.
카드를 `100%` 너비로 바꾸면 콘텐츠 하나에 화면 전체 폭을 사용할 수 있습니다.
### 3) 핵심 흐름 재구성
두 열에서 한 열로 바뀌는 과정은 단순합니다.
| 구분 | 넓은 화면 | 좁은 화면 |
|---|---|---|
| 카드 너비 | `50%` | `100%` |
| 한 행의 카드 수 | 2개 | 1개 |
| 읽기 흐름 | 좌→우, 다음 행 | 위→아래 |
| 카드 높이 | 비교적 낮게 | 콘텐츠에 맞춰 재검토 |
HTML 순서를 올바르게 작성했다면 CSS가 한 열로 바뀌었을 때도 읽는 순서가 자연스럽습니다.
### 4) 쉬운 예시
사진 앨범을 펼쳤을 때는 한 면에 사진 두 장을 나란히 볼 수 있습니다.
휴대전화에서는 한 장씩 크게 넘겨 보는 편이 사진과 설명을 이해하기 쉽습니다.
### 5) 코드 예시
```html
<main class="card-list">
  <article class="card">첫 번째 기록</article>
  <article class="card">두 번째 기록</article>
  <article class="card">세 번째 기록</article>
  <article class="card">네 번째 기록</article>
</main>
```
```css
.card-list::after {
  display: block;
  clear: both;
  content: "";
}
.card {
  float: left;
  width: 50%;
  min-height: 240px;
}
@media (max-width: 800px) {
  .card {
    width: 100%;
    min-height: 320px;
  }
}
```
### 6) 헷갈리는 점
카드 너비를 `100%`로 바꿨다고 내부 이미지가 자동으로 알맞게 줄어드는 것은 아닙니다.
이미지에도 `max-width: 100%` 또는 프로젝트에 맞는 너비 규칙이 있어야 컨테이너를 넘지 않습니다.
### 7) 한 줄 정리
> 본문 반응형의 기본 패턴은 카드의 의미 순서를 유지하면서 두 열을 읽기 쉬운 한 열로 바꾸는 것이다.

## 3. footer: 긴 정보를 안전하게 쌓기
### 1) 정의
footer 반응형 패턴은 넓은 화면에서 좌우로 나뉜 보조 정보를 좁은 화면에서 위아래로 배치하고 중앙 정렬하는 방식입니다.
### 2) 왜 필요한가
하단에는 짧은 링크뿐 아니라 설명, 정책 안내처럼 길이가 달라질 수 있는 텍스트가 들어갑니다.
고정 높이와 `50%` 열을 유지하면 문장이 겹치거나 영역 밖으로 잘릴 수 있습니다.
### 3) 핵심 흐름 재구성
footer를 전환할 때는 세 가지를 함께 봅니다.
1. 각 정보 블록의 너비를 `100%`로 바꾼다.
2. 왼쪽·오른쪽 정렬을 좁은 화면에 맞게 다시 지정한다.
3. 고정 높이보다 패딩과 콘텐츠 흐름으로 전체 높이를 확보한다.
```css
@media (max-width: 800px) {
  .footer-primary,
  .footer-secondary {
    width: 100%;
  }
  .site-footer {
    height: auto;
    padding: 24px 16px;
  }
  .site-footer p {
    margin: 0;
    text-align: center;
  }
}
```
### 4) 쉬운 예시
긴 표지판 두 개를 좁은 벽에 억지로 나란히 붙이면 글자가 작아집니다.
한 장을 위에, 다른 한 장을 아래에 두면 각 문장을 충분한 폭으로 읽을 수 있습니다.
### 5) 코드 예시
```html
<footer class="site-footer">
  <div class="footer-primary"><p>새 글 알림을 받아 보세요.</p></div>
  <div class="footer-secondary"><p>문의는 도움말 메뉴에서 확인할 수 있습니다.</p></div>
</footer>
```
```css
.footer-primary,
.footer-secondary {
  float: left;
  width: 50%;
}
@media (max-width: 800px) {
  .footer-primary,
  .footer-secondary {
    float: none;
    width: 100%;
  }
  .footer-secondary {
    margin-top: 16px;
  }
}
```
### 6) 헷갈리는 점
`float`를 끝내고 싶을 때 `float: none`은 유효하지만, 모든 속성을 `none`으로 초기화할 수는 없습니다.
예를 들어 텍스트 정렬은 `text-align: center`, 패딩 제거는 `padding: 0`, 높이 자동 계산은 `height: auto`처럼 각 속성에 맞는 값을 사용해야 합니다.
### 7) 한 줄 정리
> footer는 열을 쌓는 것과 함께 정렬, 간격, 콘텐츠 높이를 다시 계산해야 긴 정보도 안전하게 보인다.

## 코드로 보기 — 세 구역이 함께 변하는 페이지
```html
<div class="page-shell">
  <header class="top">
    <h1>작업 노트</h1>
    <nav><a href="#today">오늘</a><a href="#week">이번 주</a></nav>
  </header>
  <main class="cards">
    <article>계획 세우기</article>
    <article>진행 기록</article>
    <article>배운 점</article>
    <article>다음 행동</article>
  </main>
  <footer class="bottom">
    <p>기록은 매일 갱신됩니다.</p>
    <p>키보드와 터치로 메뉴를 사용할 수 있습니다.</p>
  </footer>
</div>
```
```css
* { box-sizing: border-box; }
body { margin: 0; }
.page-shell { width: 960px; max-width: 100%; margin: 0 auto; }
.top, .cards, .bottom { display: flow-root; }
.top h1, .top nav { float: left; width: 50%; min-height: 80px; }
.top nav a { display: inline-block; width: 50%; padding: 28px 8px; text-align: center; }
.cards article { float: left; width: 50%; min-height: 220px; padding: 24px; }
.bottom p { float: left; width: 50%; margin: 0; padding: 24px; }
@media (max-width: 800px) {
  .top h1, .top nav, .cards article, .bottom p {
    float: none;
    width: 100%;
  }
  .top h1 { margin: 0; padding: 20px; text-align: center; }
  .cards article { min-height: 280px; }
  .bottom { padding: 20px 0; }
  .bottom p { padding: 8px 16px; text-align: center; }
}
```
### 코드 목적
header, main, footer의 가로 열을 한 breakpoint에서 세로 흐름으로 전환합니다.
### 코드 흐름
1. 컨테이너는 최대 `960px`이지만 화면보다 커지지 않게 한다.
2. 넓은 화면에서 각 구역의 자식을 `50%` 열로 배치한다.
3. `800px` 이하에서는 모든 열의 float를 해제하고 `100%`로 바꾼다.
4. 상단 제목과 하단 문장을 중앙 정렬한다.
5. 본문 카드는 한 열에서 읽기 좋은 최소 높이로 조정한다.
### 예상 렌더링
```text
960px 화면: header 2열 / main 2열 / footer 2열
800px 화면: header 1열 2행 / main 1열 4행 / footer 1열 2행
카드 너비: 480px 안팎 → 화면 가용 너비 100%
footer 문장: 좌우 배치 → 위아래 중앙 정렬
```
### 실행 결과 해석
넓은 화면에서는 두 개의 `50%` 열이 한 행을 채웁니다.
`800px` 이하에서는 각 요소가 `100%`가 되고 float가 해제되어 HTML 순서대로 위에서 아래로 쌓입니다.
컨테이너의 `max-width: 100%` 덕분에 고정 기준값 `960px`이 작은 viewport 밖으로 넘지 않습니다.
### 실무 연결
콘텐츠 목록, 회사 소개, 포트폴리오 카드, 간단한 관리 화면처럼 상단 탐색·본문 카드·하단 안내로 구성된 페이지의 반응형 골격으로 활용할 수 있습니다.
## 직접 해보기
1. 넓은 화면에서 `50%`인 카드 네 개는 한 행에 몇 개씩 배치되나요?
2. 모바일에서 footer 문장이 컨테이너 밖으로 잘릴 때 너비 외에 어떤 속성을 우선 확인해야 하나요?
3. header의 두 자식을 `100%`로 바꿨는데 상단 아래 콘텐츠가 겹친다면 어떤 높이와 흐름 문제를 점검해야 하나요?
<details>
<summary>정답 보기</summary>

1. 한 행에 두 개씩 배치되어 전체 두 행이 됩니다.
2. 고정 `height`, 큰 좌우 `padding`, float 해제 여부, 긴 문자열의 줄바꿈을 확인합니다. 콘텐츠 길이에 따라 높이가 늘어나도록 `height: auto`도 검토합니다.
3. 기존 header 높이가 한 줄 기준으로 고정되어 있는지, float 자식을 부모가 포함하는지 확인합니다. 두 줄 높이를 확보하거나 자동 높이와 포함 관계를 설정해야 합니다.

</details>
## 헷갈리기 쉬운 포인트
| 헷갈리는 개념 | 차이 |
|---|---|
| 너비 변경 vs 레이아웃 전환 | 숫자만 줄이는 것 vs 열 수와 읽는 방향을 바꾸는 것 |
| `height: auto` vs 고정 높이 | 콘텐츠에 따라 늘어남 vs 지정한 높이를 유지해 넘침 가능 |
| `float: none` vs `clear: both` | 자기 요소의 부유를 해제 vs 앞선 부유 요소 옆에 놓이지 않게 함 |
| `width: 100%` vs `max-width: 100%` | 현재 컨테이너 폭을 사용 vs 원래 너비를 유지하되 컨테이너보다 커지지 않음 |
## 연결되는 개념
- 이전에 알면 좋은 개념: [반응형 웹과 viewport](05-responsive-web-and-viewport.md)
- 다음에 이어지는 개념: [반응형 모션 워크숍](07-responsive-motion-workshop.md)
- 함께 보면 좋은 키워드: `document-order`, `float`, `content-flow`
## 셀프 체크
- [ ] header를 한 줄에서 두 줄로 바꿀 때 필요한 속성을 설명할 수 있다.
- [ ] 카드 `50%`와 `100%`가 만드는 열 수를 예상할 수 있다.
- [ ] footer의 고정 높이가 위험한 이유를 말할 수 있다.
- [ ] float를 사용하는 반응형 레이아웃의 해제 지점을 찾을 수 있다.
- [ ] 경계값 앞뒤에서 각 구역의 흐름을 검증할 수 있다.
### 복습 질문 및 답변
**Q1. 모바일에서 모든 텍스트 크기를 줄이는 것이 반응형 설계의 핵심인가요?**
<details>
<summary>답</summary>

아닙니다. 핵심은 좁은 화면에서도 콘텐츠의 의미 순서와 사용성을 유지하는 것입니다. 열을 행으로 바꾸고 충분한 너비와 간격을 확보하는 일이 먼저입니다.

</details>
**Q2. main 카드의 HTML 순서가 왜 중요한가요?**
<details>
<summary>답</summary>

두 열이 한 열로 바뀌면 요소는 문서 순서대로 세로 배치됩니다. HTML 순서가 의미와 다르면 모바일에서 읽는 흐름도 어색해지므로 구조 단계에서 올바르게 정해야 합니다.

</details>
**Q3. 페이지 전체를 반응형으로 바꿀 때 어떤 순서로 검사하면 좋을까요?**
<details>
<summary>답</summary>

먼저 header의 로고와 메뉴가 겹치지 않는지 보고, main의 카드 열과 이미지 넘침을 확인합니다. 마지막으로 footer의 긴 문장, 정렬, 높이를 점검한 뒤 breakpoint 바로 앞뒤에서도 같은 검사를 반복합니다.

</details>
## 한 줄 정리
> header·main·footer의 반응형 전환은 각 요소의 너비만 바꾸는 일이 아니라 문서 순서, 높이, 정렬, 콘텐츠 흐름을 좁은 화면에 맞게 다시 설계하는 과정이다.
