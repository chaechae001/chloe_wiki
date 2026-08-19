# Flexbox 반응형 레이아웃

Flexbox는 한 방향으로 흐르는 요소의 크기와 정렬을 부모에서 선언적으로 제어합니다.

**키워드:** flex container, main axis, cross axis, flex-grow, gap

## 핵심 포인트

- `display: flex`를 선언한 요소가 컨테이너가 됩니다.
- `flex-direction`이 주축을 결정합니다.
- `justify-content`는 주축, `align-items`는 교차축 정렬입니다.
- `flex` 축약 속성은 grow, shrink, basis를 묶습니다.
- 중첩 Flexbox로 헤더, 카드, 폼 같은 복합 UI를 구성합니다.

## 축과 정렬

```css
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}
```

목적은 로고와 작업 버튼을 양 끝에 놓고 세로 중앙을 맞추는 것입니다. 기본 `row`의 주축은 가로이므로 `space-between`이 좌우 간격을 만들고, 교차축에서는 `align-items`가 높이를 정렬합니다.

## 고정 영역과 유동 영역

```css
.layout { display: flex; min-height: 100vh; }
.sidebar { flex: 0 0 16rem; }
.content { flex: 1 1 auto; min-width: 0; }
```

사이드바는 16rem으로 고정하고 본문은 남은 공간을 차지합니다. `min-width: 0`은 긴 콘텐츠가 flex item의 최소 너비를 강제해 넘치는 문제를 막습니다. 관리 화면, 메일함, 문서 앱에 자주 쓰입니다.

## 작은 화면 대응

```css
.cards { display: flex; flex-wrap: wrap; gap: 1rem; }
.card { flex: 1 1 16rem; }
```

각 카드는 16rem을 선호하지만 공간이 부족하면 다음 줄로 이동합니다. 명시적인 화면 폭보다 콘텐츠가 필요한 최소 폭을 기준으로 반응합니다.

| `justify-content` | `align-items` |
|---|---|
| 주축의 남는 공간 배치 | 교차축의 item 정렬 |
| direction에 따라 축이 바뀜 | stretch가 기본값 |
| 컨테이너에 선언 | 컨테이너에 선언 |

## 연결해서 보기

- 박스 크기와 단위는 [CSS 레이아웃 기초](02-css-layout-foundations.md)를 참고합니다.
- 배치 스타일의 관리 방식은 [Sass와 styled-components](04-sass-and-styled-components.md)로 이어집니다.

## 직접 해보기

1. 로고, 검색창, 버튼을 한 줄로 배치하세요.
2. 검색창만 남은 너비를 차지하게 만드세요.
3. 16rem보다 좁아지면 카드가 줄바꿈되게 하세요.

<details>
<summary>답</summary>

부모에 `display: flex; align-items: center; gap: 1rem`, 검색 영역에 `flex: 1`, 카드 부모에 `flex-wrap: wrap`, 카드에 `flex: 1 1 16rem`을 적용합니다.

</details>

## 점검 목록

- [ ] 주축과 교차축을 구분한다.
- [ ] 부모와 자식 속성을 구분한다.
- [ ] gap으로 요소 간 간격을 관리한다.
- [ ] 유동 item의 overflow를 확인한다.
- [ ] 작은 화면에서 줄바꿈을 검증한다.

## 복습 질문 및 답변

### 1. row에서 주축은 어느 방향인가?

<details>
<summary>답</summary>

일반적인 좌→우 문서에서는 가로 방향입니다.

</details>

### 2. `flex: 1`의 역할은?

<details>
<summary>답</summary>

item이 남은 공간을 나누어 차지하도록 grow를 활성화합니다.

</details>

### 3. `min-width: 0`이 필요한 이유는?

<details>
<summary>답</summary>

긴 콘텐츠 때문에 flex item이 줄어들지 못하고 컨테이너 밖으로 넘치는 현상을 막기 위해서입니다.

</details>

## 정리

Flexbox는 방향을 먼저 정하고 주축·교차축 정렬과 item 크기 규칙을 차례로 설정하면 쉽게 설계할 수 있습니다.
