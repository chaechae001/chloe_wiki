# CSS Transition과 Hover — 상태 변화를 부드러운 피드백으로 연결하기
> 값이 즉시 바뀌면 사용자는 무엇이 달라졌는지 놓치기 쉽다.
> `transition`은 시작 상태와 도착 상태 사이를 이어, 메뉴와 카드의 반응을 이해하기 쉬운 피드백으로 만든다.

`transition` · `duration` · `timing-function` · `delay` · `:hover`
## 핵심요약
- `transition`은 CSS 속성값이 바뀔 때 중간 단계를 자동으로 계산한다.
- `property`는 대상, `duration`은 소요 시간, `timing-function`은 속도 곡선, `delay`는 시작 대기 시간을 정한다.
- `:hover`는 포인터가 요소 위에 있는 상태를 선택하며, 도착 상태를 만드는 데 사용할 수 있다.
- 전환 선언은 기본 상태에 두어야 진입과 이탈 양쪽 변화가 모두 부드럽다.
- `all`보다 실제로 변하는 속성을 명시하면 의도와 성능을 파악하기 쉽다.
---
## 1. Transition의 네 가지 구성 요소
### 1) 정의
`transition`은 한 CSS 속성의 시작값이 다른 값으로 바뀔 때 그 사이를 일정 시간 동안 이어 주는 속성이다.
직접 장면을 여러 개 작성하는 것이 아니라, 브라우저가 두 상태 사이의 중간값을 계산한다.
### 2) 왜 필요한가
버튼 색이 한순간에 바뀌어도 기능은 동작하지만 변화의 원인을 알아차리기 어렵다.
짧은 전환을 더하면 사용자가 자신의 포인터 이동과 화면 반응을 연결할 수 있다.
전환은 장식만이 아니라 “이 요소와 상호작용할 수 있다”는 상태 피드백이 된다.
### 3) 핵심 흐름
전환은 네 가지 질문으로 나누면 읽기 쉽다.

| 속성 | 답하는 질문 | 예시 값 | 의미 |
|---|---|---|---|
| `transition-property` | 무엇이 변하는가? | `color` | 글자색만 전환 |
| `transition-duration` | 얼마나 걸리는가? | `300ms` | 0.3초 동안 진행 |
| `transition-timing-function` | 어떤 속도로 변하는가? | `ease-out` | 끝부분에서 감속 |
| `transition-delay` | 언제 시작하는가? | `100ms` | 0.1초 기다린 뒤 시작 |
다음 두 선언은 같은 뜻이다.
```css
.button {
  transition-property: background-color;
  transition-duration: 300ms;
  transition-timing-function: ease-out;
  transition-delay: 100ms;
}
```
```css
.button {
  transition: background-color 300ms ease-out 100ms;
}
```
축약형에서 시간값이 두 개라면 첫 번째가 `duration`, 두 번째가 `delay`다.
시간은 `s` 또는 `ms`로 표현하며 `0.3s`와 `300ms`는 같다.
지속 시간이 `0s`라면 전환은 사실상 즉시 일어난다.
### 4) 쉬운 예시
엘리베이터 문이 닫히는 장면을 떠올려 보자.
문 자체는 `property`, 닫히는 데 걸리는 시간은 `duration`, 출발과 감속의 느낌은 `timing-function`, 안내음 뒤 기다리는 시간은 `delay`에 해당한다.
### 5) 코드 예시
```html
<button class="save-button">저장</button>
```
```css
.save-button {
  padding: 12px 20px;
  border: 0;
  border-radius: 8px;
  color: #ffffff;
  background-color: #315b8a;
  transition: background-color 250ms ease-out 50ms;
}
.save-button:hover {
  background-color: #183b63;
}
```
### 6) 결과 해석과 헷갈리는 점
포인터를 올리고 50ms가 지나면 배경색이 250ms 동안 `#315b8a`에서 `#183b63`으로 바뀐다.
전체 반응은 대기 50ms와 진행 250ms를 합쳐 약 300ms 후 도착 상태에 이른다.
- `duration`은 변화가 진행되는 시간이고 `delay`는 시작 전 아무 변화 없이 기다리는 시간이다.
- `transition`만 선언해도 상태를 바꾸는 별도 규칙이 없으면 화면은 변하지 않는다.
- 숫자처럼 중간값을 계산할 수 있는 속성은 부드럽지만 `display: none`과 `display: block`은 같은 방식으로 보간되지 않는다.
### 7) 한 줄 정리
> 전환은 대상·시간·속도 곡선·대기 시간이라는 네 질문으로 설계한다.
---
## 2. timing-function으로 움직임의 리듬 만들기
### 1) 정의
`transition-timing-function`은 전체 진행 시간 안에서 변화 속도가 어떻게 달라질지를 정한다.
도착 시간은 같아도 가속과 감속 방식에 따라 빠르거나 무겁게 느껴질 수 있다.
### 2) 왜 필요한가
모든 상태 변화가 일정한 속도로 움직이면 기계적으로 보일 수 있다.
반대로 짧은 버튼 피드백에 과한 가감속을 주면 반응이 늦게 느껴진다.
화면의 역할에 맞는 리듬을 고르면 같은 `300ms`도 더 자연스럽게 전달된다.
### 3) 핵심 흐름
| 값 | 속도 느낌 | 어울리는 상황 |
|---|---|---|
| `linear` | 처음부터 끝까지 일정 | 반복 이동, 속도 비교 실험 |
| `ease` | 부드럽게 출발하고 감속 | 일반적인 상태 변화 |
| `ease-in` | 천천히 출발 | 화면에서 사라지는 요소 |
| `ease-out` | 빠르게 출발해 감속 | 사용자의 입력에 즉시 반응하는 요소 |
| `ease-in-out` | 출발과 도착이 모두 부드러움 | 패널 크기나 위치 변화 |
`linear`는 진행 시간의 절반이 지났을 때 변화량도 정확히 절반이다.
다른 곡선은 같은 시점에 더 많이 또는 더 적게 진행될 수 있지만 최종값과 전체 시간은 같다.
### 4) 쉬운 예시
정류장 사이를 달리는 버스는 출발할 때 가속하고 도착 전에 감속한다.
이 리듬은 `ease-in-out`에 가깝고, 컨베이어 벨트처럼 일정하게 움직이는 장치는 `linear`에 가깝다.
### 5) 코드 예시
```css
.linear-chip {
  transition: transform 600ms linear;
}
.ease-chip {
  transition: transform 600ms ease-out;
}
.linear-chip:hover,
.ease-chip:hover {
  transform: translateX(120px);
}
```
### 6) 결과 해석과 헷갈리는 점
두 칩은 모두 600ms 뒤 오른쪽 120px 지점에 도착한다.
`linear-chip`은 매 구간을 같은 속도로 지나고, `ease-chip`은 초반에 더 빠르게 이동한 뒤 도착점 가까이에서 느려진다.
- 속도 곡선은 총 이동 거리나 최종값을 바꾸지 않는다.
- `linear`가 항상 빠른 것은 아니다. 전체 시간은 `duration`이 결정한다.
- 짧은 피드백은 대기 시간이 길면 고장 난 것처럼 느껴질 수 있으므로 `delay`를 목적 없이 붙이지 않는다.
### 7) 한 줄 정리
> 타이밍 함수는 도착점이 아니라 도착하는 과정의 리듬을 설계한다.
---
## 3. :hover로 시작 상태와 도착 상태 연결하기
### 1) 정의
`:hover`는 포인터가 요소의 영역 위에 놓인 상태를 선택하는 가상 클래스다.

기본 선택자가 시작 상태를, `:hover` 선택자가 도착 상태를 담당하면 전환이 두 값을 연결한다.
### 2) 왜 필요한가
메뉴 링크의 색, 카드의 배경, 이미지의 크기를 바꾸면 사용자가 현재 가리키는 대상을 빠르게 구분할 수 있다.

특히 반복되는 카드 목록에서는 작고 일관된 피드백이 선택 실수를 줄인다.
### 3) 핵심 흐름
```css
.card {
  background-color: #f3f6fb;
  transform: scale(1);
  transition:
    background-color 300ms ease-out,
    transform 300ms ease-out;
}

.card:hover {
  background-color: #dceaff;
  transform: scale(1.03);
}
```
전환 선언은 `.card:hover`가 아니라 기본 `.card`에 있다.

그래서 포인터가 들어갈 때뿐 아니라 나갈 때도 300ms 동안 원래 상태로 돌아온다.

여러 속성은 쉼표로 나누어 각자의 시간과 속도 곡선을 지정할 수 있다.
### 4) 쉬운 예시
복도 조명이 사람을 감지하면 서서히 밝아지고, 사람이 지나가면 서서히 어두워지는 장면과 같다.

센서가 감지한 상태는 `:hover`, 밝기 변화의 규칙은 `transition`이다.
### 5) 코드 예시
```css
.menu-link {
  color: #27364a;
  transition: color 300ms ease-out;
}

.menu-link:hover {
  color: #8a4f63;
}

.card-image {
  transform: scale(1);
  transition: transform 300ms ease-out;
}

.card-image:hover {
  transform: scale(1.08);
}
```
### 6) 결과 해석과 헷갈리는 점
메뉴 글자는 300ms 동안 두 색 사이를 이동하고, 이미지는 100%에서 108%까지 확대된다.

이미지의 실제 배치 너비를 바꾼 것이 아니므로 주변 카드가 밀려나지 않는다.
- `.menu-link :hover`처럼 공백을 넣으면 링크 자체가 아니라 그 안의 hover 상태인 후손을 찾는다.
- 포인터가 없는 환경에서는 hover가 핵심 기능의 유일한 단서가 되지 않도록 해야 한다.
- `transition: all`은 예상하지 못한 속성까지 전환할 수 있으므로 `color`, `background-color`, `transform`처럼 대상을 쓰는 편이 명확하다.
### 7) 한 줄 정리
> 기본 상태에 전환을 두고 `:hover`에 도착값을 지정하면 진입과 이탈이 모두 자연스럽다.
---
## 코드로 보기 — 메뉴와 카드의 반응 완성하기
```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>Transition 예제</title>
  <style>
    body { font-family: sans-serif; padding: 32px; color: #243247; }
    .menu { display: flex; gap: 20px; margin-bottom: 28px; }
    .menu a {
      color: #315b8a;
      text-decoration: none;
      transition: color 300ms ease-out;
    }
    .menu a:hover { color: #8a4f63; }
    .card {
      width: 240px;
      padding: 24px;
      border-radius: 12px;
      background-color: #edf4ff;
      transform: scale(1);
      transition:
        background-color 300ms ease-out,
        transform 300ms ease-out;
    }
    .card:hover {
      background-color: #d6e7ff;
      transform: scale(1.05);
    }
  </style>
</head>
<body>
  <nav class="menu">
    <a href="#guide">이용 안내</a>
    <a href="#news">새 소식</a>
  </nav>
  <article class="card">
    <h2>오늘의 추천</h2>
    <p>포인터를 올려 상태 변화를 확인해 보세요.</p>
  </article>
</body>
</html>
```
### 코드 목적
메뉴 링크의 색 변화와 카드의 배경·크기 변화를 각각 명시적인 전환 대상으로 설계한다.
### 코드 흐름
1. 메뉴 링크의 기본색과 hover 색을 정한다.
2. 링크의 `color`만 300ms 동안 전환한다.
3. 카드의 기본 배경과 `scale(1)`을 시작값으로 둔다.
4. hover 상태에서 배경색과 배율을 동시에 바꾼다.
### 예상 렌더링
```text
메뉴: 포인터 진입 후 300ms 동안 #315b8a → #8a4f63
카드 배경: 포인터 진입 후 300ms 동안 #edf4ff → #d6e7ff
카드 크기: 포인터 진입 후 300ms 동안 100% → 105%
포인터 이탈: 세 속성이 같은 시간 동안 기본값으로 복귀
```
### 실행 결과 해석
메뉴 링크는 크기나 배치를 건드리지 않고 색만 바뀐다.

카드는 300ms 동안 5% 커지며 배경도 함께 밝아져 현재 선택 가능한 영역을 강조한다.

전환 대상이 명시되어 있으므로 나중에 테두리나 여백을 수정해도 뜻하지 않게 애니메이션되지 않는다.
### 실무 연결
상품 목록, 관리자 화면의 작업 카드, 문서 탐색 메뉴처럼 반복 요소가 많은 화면에서 hover 피드백은 현재 초점을 빠르게 알려 준다.

디자인 시스템에서는 전환 시간과 타이밍 함수를 공통 값으로 정해 두면 여러 컴포넌트가 같은 반응 속도를 유지한다.
---
## 직접 해보기
1. `transition: width 2s linear 1s;`에서 변화가 시작되는 시점과 완료되는 시점을 설명해 보자.
2. 카드가 들어갈 때는 부드럽지만 포인터를 빼는 순간 바로 원래대로 돌아온다면 전환 선언을 어디로 옮겨야 하는가?
3. 메뉴 글자색만 바꾸려는데 `transition: all 300ms;`가 쓰였다면 어떻게 개선할 수 있는가?

<details>
<summary>정답 보기</summary>
1. 상태가 바뀐 뒤 1초 동안 기다리고, 그다음 2초 동안 일정한 속도로 너비가 변한다. 따라서 도착값에는 상태 변경 후 약 3초에 도달한다.
2. `:hover` 규칙이 아니라 기본 상태 선택자로 옮긴다. 기본 상태에 선언해야 진입과 이탈 양쪽 값 변화에 모두 전환이 적용된다.
3. `transition: color 300ms;`처럼 실제로 바뀌는 속성을 명시한다. 의도가 분명해지고 다른 스타일 변경이 뜻밖에 전환되는 일을 줄인다.
</details>
## 헷갈리기 쉬운 포인트
| 헷갈리는 개념 | 차이 |
|---|---|
| `duration` vs `delay` | 앞은 변화가 진행되는 시간, 뒤는 변화가 시작되기 전 대기 시간 |
| `linear` vs `ease-out` | 앞은 전 구간 속도가 일정하고, 뒤는 빠르게 출발해 도착점에서 감속한다 |
| `transition` vs `:hover` | 앞은 두 값 사이의 변화 방법, 뒤는 포인터가 올라간 상태를 선택하는 조건 |
| 기본 상태 선언 vs hover 상태 선언 | 기본 상태에 전환을 두면 진입·이탈 모두 부드럽고, hover에만 두면 이탈 시 즉시 돌아갈 수 있다 |
| 특정 속성 vs `all` | 특정 속성은 의도가 명확하고, `all`은 예상하지 못한 변화까지 포함할 수 있다 |
## 연결되는 개념
- 이전에 알면 좋은 개념: [CSS Transform 기초](01-transform-basics.md)
- 다음에 이어지는 개념: [CSS Keyframe Animation](03-css-keyframe-animation.md)
- 전체 흐름 다시 보기: [움직이는 웹사이트와 반응형 웹 학습 지도](OVERVIEW.md)
- 함께 보면 좋은 키워드: `상태 변화`, `보간`, `상호작용 피드백`
## 셀프 체크
- [ ] 전환의 네 가지 구성 요소를 설명할 수 있다.
- [ ] 축약형에서 지속 시간과 대기 시간을 구분할 수 있다.
- [ ] 대표 타이밍 함수의 속도 느낌을 비교할 수 있다.
- [ ] 기본 상태와 hover 상태를 나누어 작성할 수 있다.
- [ ] `all` 대신 전환 대상을 구체적으로 지정할 수 있다.
### 복습 질문 및 답변
**Q1. 기본 — `transition-property`는 무엇을 정하는가?**
<details>
<summary>답</summary>
값이 바뀔 때 중간 단계를 계산할 CSS 속성을 정한다. 예를 들어 `color`를 쓰면 글자색 변화에만 전환이 적용된다.
</details>
**Q2. 이해 확인 — `300ms` 전환에서 `linear`와 `ease-out`의 공통점과 차이점은?**
<details>
<summary>답</summary>
두 방식 모두 300ms 후 같은 최종값에 도착한다. `linear`는 진행 속도가 일정하고, `ease-out`은 초반에 빠르다가 끝에서 감속한다.
</details>
**Q3. 응용 — 카드의 배경색과 확대 효과를 서로 다른 시간으로 움직이게 하려면 어떻게 작성하는가?**
<details>
<summary>답</summary>
`transition: background-color 200ms ease-out, transform 350ms ease-out;`처럼 쉼표로 두 전환을 분리한다. 각 속성에 독립적인 지속 시간과 속도 곡선을 줄 수 있다.
</details>
## 한 줄 정리
> 좋은 전환은 값의 변화만 보여 주는 장식이 아니라, 무엇이 반응했고 어디로 상태가 바뀌는지를 시간과 리듬으로 설명하는 인터페이스 언어다.
