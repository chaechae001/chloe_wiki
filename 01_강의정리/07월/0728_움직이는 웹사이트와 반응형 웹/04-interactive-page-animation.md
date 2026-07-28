# 인터랙티브 페이지 애니메이션 — 메뉴와 콘텐츠에 자연스러운 반응 만들기

> 움직임은 많다고 좋은 것이 아니라 사용자의 행동에 알맞은 피드백을 줄 때 가치가 있습니다. 메뉴의 색 변화와 카드의 배경·이미지 확대를 작은 규칙으로 나누면 일관되고 수정하기 쉬운 상호작용을 만들 수 있습니다.

`:hover` · `transition` · `transform` · `scale` · `interaction-feedback`

## 핵심요약

- 상호작용 효과는 기본 상태, 작동 조건, 목표 상태, 전환 방식의 네 부분으로 나누어 설계한다.
- 메뉴 링크는 `:hover` 상태에서 글자색을 바꾸고 기본 상태에 `transition`을 선언한다.
- 콘텐츠 카드는 배경색 전환과 이미지 확대를 서로 다른 요소에 맡기면 구조가 명확하다.
- `transform: scale()`은 레이아웃의 자리 크기를 다시 계산하지 않고 시각적으로 요소를 확대한다.
- 전체 속성을 뜻하는 `all`보다 실제로 변하는 속성을 적으면 움직임의 범위와 의도가 선명해진다.

---

## 1. 상호작용 효과를 네 단계로 설계하기

### 1) 정의

인터랙티브 애니메이션은 사용자 행동에 반응해 화면 상태를 바꾸는 움직임입니다.
CSS에서는 기본 스타일, `:hover` 같은 조건, 조건에서 달라질 목표값, 두 상태를 잇는 `transition`으로 구성할 수 있습니다.

### 2) 왜 필요한가

링크에 마우스를 올렸는데 아무 변화가 없으면 사용자는 클릭 가능한지 판단하기 어렵습니다. 짧은 색 변화나 크기 변화는 현재 가리키는 대상을 알려 주고, 인터페이스가 입력을 인식했다는 피드백을 제공합니다.

### 3) 핵심 흐름 재구성

상호작용을 만들 때 다음 네 질문을 순서대로 답합니다.

1. 기본 상태는 어떤 모습인가?
2. 어떤 조건에서 반응할 것인가?
3. 반응 뒤 어떤 값으로 바뀌는가?
4. 그 변화가 얼마 동안 어떤 속도로 이어지는가?

`transition`은 변화가 시작되는 `:hover` 규칙이 아니라 기본 규칙에 둡니다. 그래야 마우스를 올릴 때와 뗄 때 모두 같은 시간으로 부드럽게 전환됩니다.

### 4) 쉬운 예시

엘리베이터 버튼을 누르면 불이 켜지는 것처럼, 메뉴 색 변화는 “이 항목을 가리키고 있다”는 짧은 응답입니다.
버튼 자체보다 응답이 늦거나 지나치게 화려하면 오히려 조작이 답답하게 느껴질 수 있습니다.

### 5) 코드 예시

```html
<nav aria-label="콘텐츠 메뉴">
  <a class="menu-link" href="#guide">가이드</a><a class="menu-link" href="#examples">예제</a>
  <a class="menu-link" href="#notes">기록</a>
</nav>
```

```css
.menu-link {
  display: inline-block; padding: 10px 14px;
  color: #2d3142; text-decoration: none;
  transition: color 300ms;
}
.menu-link:hover { color: #6c5ce7; }
```

### 6) 헷갈리는 점

- `.menu-link:hover`는 해당 링크 자체의 hover 상태입니다.
- `.menu-link :hover`처럼 공백을 넣으면 링크 내부의 후손 중 hover된 요소를 찾는 다른 선택자가 됩니다.
- `transition`은 목표값을 만들지 않습니다. 기본 상태와 hover 상태에 서로 다른 값이 있어야 합니다.

### 7) 한 줄 정리

> 상호작용 효과는 상태 차이를 먼저 만들고 기본 규칙의 transition으로 그 차이를 이어 준다.

---

## 2. 카드의 배경과 이미지를 역할별로 움직이기

### 1) 정의

하나의 카드 안에서도 배경과 이미지는 서로 다른 요소입니다.
카드에는 배경색 전환을, 이미지에는 확대 변형을 적용하면 각 효과의 대상과 책임이 분명해집니다.

### 2) 왜 필요한가

카드 전체를 확대하면 주변 요소와 겹치거나 글자까지 커질 수 있습니다. 이미지에만 `scale()`을 적용하고 카드에는 은은한 색 변화만 주면 정보 구조를 유지하면서 피드백을 만들 수 있습니다.

### 3) 핵심 흐름 재구성

카드 상호작용은 다음처럼 나눌 수 있습니다.

| 대상 | 기본 상태 | hover 상태 | 전환 속성 |
|---|---|---|---|
| 카드 | 어두운 배경 | 밝은 배경 | `background-color` |
| 이미지 | 배율 `1` | 배율 `1.08` | `transform` |
| 메뉴 링크 | 기본 글자색 | 강조 글자색 | `color` |

각 요소의 기본 규칙에 실제로 바뀌는 속성만 지정합니다.

```css
.card {
  background-color: #343a5f;
  transition: background-color 300ms;
}

.card:hover {
  background-color: #555d8f;
}

.card img {
  transform: scale(1);
  transition: transform 300ms;
}

.card:hover img {
  transform: scale(1.08);
}
```

### 4) 쉬운 예시

서점에서 책 한 권을 집어 들면 진열대 전체가 커지는 것이 아니라 선택한 책만 눈앞으로 가까워집니다.
카드 배경은 선택 범위를 표시하고 이미지 확대는 시선을 핵심 콘텐츠로 모으는 역할을 합니다.

### 5) 코드 예시

```html
<article class="card">
  <a href="#detail">
    <div class="card-media"><img src="https://picsum.photos/480/280" alt="잔잔한 호수 풍경"></div>
    <h2>한 주의 풍경</h2>
  </a>
</article>
```

```css
.card {
  background-color: #343a5f;
  transition: background-color 300ms;
}
.card:hover { background-color: #555d8f; }
.card-media { overflow: hidden; }
.card img {
  display: block; width: 100%; transform: scale(1);
  transition: transform 300ms;
}
.card:hover img { transform: scale(1.08); }
```

`overflow: hidden`은 확대된 이미지가 카드의 미디어 영역 바깥으로 튀어나오지 않도록 잘라 줍니다.

### 6) 헷갈리는 점

- `scale(1)`은 원래 크기이고 `scale(1.08)`은 가로와 세로를 모두 1.08배로 보입니다.
- `transform`은 화면에 그려지는 모양을 바꾸지만 주변 레이아웃이 차지한 자리를 다시 계산하지 않습니다.
- `.card img:hover`는 이미지 위에 직접 마우스를 올렸을 때만 작동하지만 `.card:hover img`는 카드 안 어디를 가리켜도 이미지를 선택합니다.

### 7) 한 줄 정리

> 카드와 이미지의 움직임을 분리하면 배경 피드백과 시각적 강조를 독립적으로 조절할 수 있다.

## 코드로 보기 — 탐색 메뉴와 콘텐츠 카드 묶기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
  <title>인터랙티브 카드</title>
  <style>
    * { box-sizing: border-box; }
    body { margin: 0; color: #23263b; font-family: sans-serif; background: #f4f5fb; }
    .page-header { padding: 20px; background: #fff; } .menu { display: flex; gap: 8px; }
    .menu a {
      padding: 10px 14px; color: #343a5f; text-decoration: none;
      transition: color 300ms, background-color 300ms;
    }
    .menu a:hover { color: #fff; background-color: #6c5ce7; }
    .card-grid { display: flex; gap: 20px; padding: 28px; }
    .card {
      width: 260px; overflow: hidden; border-radius: 12px;
      background-color: #343a5f; transition: background-color 300ms;
    }
    .card:hover { background-color: #555d8f; } .card a { display: block; color: #fff; text-decoration: none; }
    .card-media { height: 150px; overflow: hidden; }
    .card img {
      display: block; width: 100%; height: 100%; object-fit: cover;
      transform: scale(1); transition: transform 300ms;
    }
    .card:hover img { transform: scale(1.08); } .card h2 { margin: 0; padding: 18px; font-size: 20px; }
  </style>
</head>
<body>
  <header class="page-header">
    <nav class="menu" aria-label="주요 메뉴">
      <a href="#latest">새 글</a><a href="#popular">인기 글</a><a href="#saved">보관함</a>
    </nav>
  </header>
  <main class="card-grid" id="latest">
    <article class="card"><a href="#read">
      <div class="card-media"><img src="https://picsum.photos/520/300" alt="숲길 풍경"></div>
      <h2>천천히 걷는 기록</h2>
    </a></article>
  </main>
</body>
</html>
```
### 코드 목적

메뉴 링크에는 색상 피드백을, 콘텐츠 카드에는 배경색과 이미지 확대를 적용해 상호작용 역할을 분리합니다.
### 코드 흐름

1. 메뉴 링크의 기본 색과 hover 목표 색을 정한다.
2. 링크의 `color`와 `background-color`만 전환 대상으로 지정한다.
3. 카드 배경은 카드 자체의 hover 상태에서 바꾼다.
4. 카드가 hover되면 후손 이미지의 `transform` 배율을 바꾼다.
5. 미디어 상자의 `overflow: hidden`으로 확대 범위를 카드 안에 제한한다.

### 예상 렌더링

```text
메뉴는 강조색으로 전환되고, 카드 배경은 밝아지며 카드 이미지는 1.08배 확대됨
```

### 실행 결과 해석

메뉴의 변화는 클릭 가능한 항목을 알려 주고, 카드의 변화는 현재 탐색 중인 콘텐츠 범위를 강조합니다. 이미지는 확대되어도 `.card-media`의 `150px` 높이 밖으로 넘치지 않으므로 주변 레이아웃의 위치는 그대로 유지됩니다.
### 실무 연결

포트폴리오 목록, 상품 카드, 콘텐츠 추천 영역처럼 반복 항목이 있는 화면에 그대로 응용할 수 있습니다.
팀 작업에서는 색상과 지속 시간을 공통 변수로 관리하면 여러 메뉴와 카드의 반응을 일관되게 유지하기 좋습니다.

---
## 직접 해보기

1. `transition`을 hover 규칙에만 선언했을 때 마우스를 뗄 때의 복귀가 자연스럽지 않을 수 있는 이유를 설명해 보세요.
2. 카드 안의 이미지만 `1.12`배로 키우고 바깥으로 넘치지 않게 하려면 어느 두 요소에 어떤 속성을 적용해야 하나요?
3. 페이지의 모든 카드가 서로 다른 전환 시간을 사용해 화면이 산만하다면 어떤 기준으로 규칙을 정리할 수 있을까요?

<details>
<summary>정답 보기</summary>

1. hover가 끝나는 순간 hover 규칙과 그 안의 전환 설정도 함께 사라집니다. 기본 규칙에 `transition`을 두면 진입과 복귀 모두 같은 전환 규칙을 사용합니다.
2. 이미지에 `transform: scale(1.12)`를 적용하고, 이미지를 감싸는 미디어 상자에 `overflow: hidden`을 적용합니다.
3. 메뉴·카드처럼 역할별 지속 시간을 정하고 실제로 바뀌는 속성만 전환 대상으로 명시합니다. 같은 역할에는 같은 시간과 속도 함수를 재사용합니다.

</details>
## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `.card:hover img` vs `.card img:hover` | 카드 전체가 hover되면 이미지 선택 vs 이미지 자체가 hover될 때만 선택 |
| `scale(1)` vs `scale(1.1)` | 원래 배율 유지 vs 가로·세로를 1.1배로 확대 |
| `transition: all` vs 속성 명시 | 모든 전환 가능 속성 대상 vs 의도한 속성만 대상 |
| 기본 상태 vs hover 상태 | 평소 보이는 값과 사용자 행동 중에 적용되는 목표값 |
| `transition` vs `transform` | 상태 사이 변화 과정을 조절 vs 요소의 이동·회전·확대 모양을 결정 |
## 연결되는 개념

- 이전에 알면 좋은 개념: [CSS 키프레임 애니메이션](03-css-keyframe-animation.md)
- 다음에 이어지는 개념: [반응형 웹과 뷰포트](05-responsive-web-and-viewport.md)
- 전체 흐름 다시 보기: [움직이는 웹사이트와 반응형 웹 학습 지도](OVERVIEW.md)
- 함께 보면 좋은 키워드: `pseudo-class`, `transition-property`, `overflow`
## 셀프 체크

- [ ] 상호작용 효과를 기본·조건·목표·전환 네 단계로 설명할 수 있다.
- [ ] 메뉴 링크의 색 전환을 직접 작성할 수 있다.
- [ ] 카드 배경과 이미지 확대 효과를 서로 다른 요소에 적용할 수 있다.
- [ ] `.card:hover img` 선택자의 의미를 설명할 수 있다.
- [ ] `all` 대신 변화할 속성을 선택할 수 있다.

### 복습 질문 및 답변

**Q1. 기본 — `:hover`는 어떤 역할을 하나요?**

<details>
<summary>답</summary>

포인터가 요소 위에 놓인 상태를 선택합니다. 이 상태에서 색, 배경, transform 같은 목표값을 지정해 사용자 행동에 대한 시각적 피드백을 만들 수 있습니다.

</details>

**Q2. 이해 확인 — `transform: scale(1.08)`을 적용했는데 주변 카드가 밀리지 않는 이유는 무엇인가요?**

<details>
<summary>답</summary>

`transform`은 요소가 화면에 그려지는 모양을 바꾸지만 문서 흐름에서 계산된 원래 자리 크기를 다시 계산하지 않기 때문입니다.

</details>

**Q3. 응용 — 카드에 마우스를 올렸을 때 배경은 바뀌지만 이미지가 확대되지 않는다면 어떤 순서로 확인해야 하나요?**

<details>
<summary>답</summary>

먼저 이미지가 실제로 카드의 후손인지 확인하고 `.card:hover img` 선택자가 맞는지 봅니다. 다음으로 hover 규칙의 `transform` 값, 기본 이미지 규칙의 `transition: transform ...`, 다른 규칙에 의해 덮어써졌는지를 차례로 확인합니다.

</details>

## 한 줄 정리

> 좋은 인터랙티브 움직임은 요소별 역할을 나누고, 사용자의 상태 변화에 필요한 속성만 짧고 일관되게 전환하는 데서 시작한다.
