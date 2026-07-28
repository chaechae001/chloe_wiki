# CSS 키프레임 애니메이션 — 시간에 따라 상태 변화를 설계하기

> 요소가 한 번 바뀌는 데서 끝나지 않고 여러 장면을 스스로 반복하게 하려면 변화의 경로와 재생 규칙을 함께 정해야 합니다. `@keyframes`와 `animation` 속성을 나누어 이해하면 움직임을 의도대로 조절할 수 있습니다.

`@keyframes` · `animation-duration` · `iteration-count` · `direction` · `fill-mode`

## 핵심요약

- `@keyframes`는 애니메이션이 진행되는 동안 요소가 거칠 상태를 정의한다.
- 요소의 `animation-name`과 `@keyframes`의 이름이 같아야 두 규칙이 연결된다.
- `duration`, `timing-function`, `delay`는 한 회차의 시간과 속도, 시작 시점을 조절한다.
- `iteration-count`와 `direction`은 반복 횟수와 각 회차의 진행 방향을 결정한다.
- `fill-mode`는 재생 전 대기 시간과 재생 종료 뒤에 어떤 스타일을 보여 줄지 결정한다.

---

## 1. `@keyframes`와 애니메이션 연결

### 1) 정의

CSS 애니메이션은 두 부분으로 구성됩니다.
`@keyframes`는 시간에 따른 변화 내용을 만들고, 요소의 `animation-*` 속성은 그 변화를 어떻게 재생할지 지정합니다.

```css
@keyframes widen {
  from { width: 120px; }
  to { width: 240px; }
}

.meter {
  animation-name: widen;
  animation-duration: 2s;
}
```

### 2) 왜 필요한가

`transition`은 hover처럼 상태가 달라지는 계기가 있어야 시작됩니다.
반면 애니메이션은 페이지가 표시된 뒤 자동으로 시작하거나 같은 변화 과정을 여러 번 반복할 수 있습니다.
로딩 표시, 주의 메시지, 장식 요소처럼 시간 자체가 진행 조건인 움직임에 알맞습니다.

### 3) 핵심 흐름 재구성

1. 움직임의 시작 상태와 끝 상태를 정한다.
2. `@keyframes` 뒤에 알아보기 쉬운 이름을 붙인다.
3. 요소의 `animation-name`에 같은 이름을 적는다.
4. 한 번 재생되는 시간인 `animation-duration`을 지정한다.
5. 필요에 따라 속도, 대기, 반복, 방향, 종료 상태를 덧붙인다.

`from`과 `to`는 각각 `0%`와 `100%`를 읽기 쉽게 표현한 문법입니다.
중간 장면이 필요하면 `50%`처럼 진행률을 직접 사용할 수 있습니다.

```css
@keyframes pulse-width {
  0% { width: 120px; }
  50% { width: 210px; }
  100% { width: 160px; }
}
```

### 4) 쉬운 예시

`@keyframes`를 영상의 장면표라고 생각할 수 있습니다.
장면표에는 시작·중간·끝 모습을 적고, `animation-*`에는 영상 이름, 재생 시간, 반복 횟수와 재생 방향을 적습니다.

### 5) 코드 예시

```html
<div class="progress" aria-label="처리 진행 중"></div>

<style>
  .progress {
    width: 80px;
    height: 12px;
    border-radius: 6px;
    background: #5b6cff;
    animation-name: grow-bar;
    animation-duration: 1.2s;
  }

  @keyframes grow-bar {
    from { width: 80px; }
    to { width: 220px; }
  }
</style>
```

브라우저는 `grow-bar`라는 같은 이름을 찾아 너비를 `80px`에서 `220px`로 바꿉니다.

### 6) 헷갈리는 점

- 이름은 따옴표 없이 작성하며 철자와 대소문자가 일치해야 합니다.
- `duration`의 기본값은 `0s`이므로 이름만 연결하면 화면에서 움직임이 보이지 않습니다.
- `@keyframes`는 변화의 내용이고, 반복 횟수와 속도는 요소 쪽 속성입니다.

### 7) 한 줄 정리

> `@keyframes`는 장면을 정의하고 `animation-*`은 그 장면을 재생하는 규칙을 정한다.

---

## 2. 시간·반복·방향을 조절하는 세부 속성

### 1) 정의

애니메이션의 움직임은 여러 세부 속성이 분담합니다.

| 속성 | 역할 | 예시 |
|---|---|---|
| `animation-name` | 사용할 키프레임 선택 | `pulse` |
| `animation-duration` | 한 회차의 재생 시간 | `1.5s` |
| `animation-timing-function` | 구간별 속도 변화 | `linear` |
| `animation-delay` | 재생 전 대기 시간 | `500ms` |
| `animation-iteration-count` | 반복 횟수 | `3`, `infinite` |
| `animation-direction` | 회차의 진행 방향 | `normal`, `alternate` |
| `animation-fill-mode` | 재생 전후 스타일 | `forwards`, `backwards` |

### 2) 왜 필요한가

같은 키프레임이라도 재생 규칙에 따라 인상이 달라집니다.
짧게 한 번 재생하면 상태 알림이 되고, 계속 왕복하면 살아 있는 장식처럼 보입니다.
따라서 “무엇이 변하는가”뿐 아니라 “언제, 몇 번, 어느 방향으로 변하는가”도 설계해야 합니다.

### 3) 핵심 흐름 재구성

`animation-direction`의 대표 값은 다음처럼 구분합니다.

| 값 | 첫 회차 | 다음 회차 |
|---|---|---|
| `normal` | 시작 → 끝 | 다시 시작 → 끝 |
| `reverse` | 끝 → 시작 | 다시 끝 → 시작 |
| `alternate` | 시작 → 끝 | 끝 → 시작 |
| `alternate-reverse` | 끝 → 시작 | 시작 → 끝 |

`alternate`는 매 회차가 끝날 때 방향을 바꾸므로 끝 위치에서 갑자기 시작 위치로 되감기는 느낌을 줄일 수 있습니다.

`animation-fill-mode`는 반복 방향과 다른 문제를 해결합니다.

- `none`: 기본값이며 재생 전후에는 원래 CSS 스타일을 사용합니다.
- `forwards`: 재생이 끝난 뒤 마지막으로 계산된 키프레임 상태를 유지합니다.
- `backwards`: 지연 시간 동안 첫 재생 방향의 시작 키프레임을 미리 적용합니다.
- `both`: `forwards`와 `backwards` 효과를 함께 적용합니다.

### 4) 쉬운 예시

진자가 좌우로 움직이는 모습을 만들 때 `normal`을 반복하면 오른쪽 끝에서 왼쪽 시작점으로 순간 이동합니다.
`alternate`를 사용하면 오른쪽에 도착한 다음 왼쪽으로 돌아오므로 실제 진자에 가까운 흐름이 됩니다.

### 5) 코드 예시

```css
.badge {
  animation-name: nod;
  animation-duration: 1.5s;
  animation-timing-function: linear;
  animation-delay: 300ms;
  animation-iteration-count: 4;
  animation-direction: alternate;
  animation-fill-mode: forwards;
}

@keyframes nod {
  from { transform: rotate(-6deg); }
  to { transform: rotate(6deg); }
}
```

네 회차가 번갈아 재생되며, 마지막 회차가 도착한 쪽의 회전 상태가 종료 후에도 유지됩니다.

### 6) 헷갈리는 점

- `infinite`는 시간 값이 아니라 반복 횟수 자리에 쓰는 키워드입니다.
- `alternate` 한 회차는 한쪽 방향 이동입니다. 왕복 전체는 두 회차를 사용합니다.
- `backwards`는 종료 뒤 처음으로 되돌리는 값이 아니라, 지연 시간에 시작 키프레임을 적용하는 값입니다.
- 밀리초 `1500ms`와 초 `1.5s`는 같은 시간입니다.

### 7) 한 줄 정리

> 시간·반복·방향·채움 속성을 분리해 보면 애니메이션의 시작부터 종료까지 예측할 수 있다.

## 코드로 보기 — 왕복하는 상태 표시 배지

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>키프레임 배지</title>
  <style>
    .status-badge {
      display: inline-block; padding: 12px 18px;
      color: #fff; border-radius: 999px; background: #5663d8;
      animation: attention 1.2s linear 200ms 4 alternate forwards;
    }
    @keyframes attention {
      from { transform: rotate(-5deg) scale(1); }
      to { transform: rotate(5deg) scale(1.08); }
    }
  </style>
</head>
<body><strong class="status-badge">새 소식</strong></body>
</html>
```
### 코드 목적

회전과 확대를 하나의 키프레임에 결합하고, 유한 반복·왕복·종료 상태 유지가 함께 작동하는 과정을 확인합니다.
### 코드 흐름

1. 배지의 기본 모양과 배경색을 지정한다.
2. `attention` 키프레임에서 회전 각도와 배율을 함께 바꾼다.
3. 한 회차를 `1.2s`로 설정하고 `200ms` 뒤에 시작한다.
4. 네 회차 동안 `alternate` 방향으로 재생한다.
5. `forwards`로 마지막 회차가 끝난 상태를 유지한다.

### 예상 렌더링

```text
0.2초 뒤 배지가 좌우로 기울며 확대·축소를 네 회차 반복하고 마지막 상태에서 멈춤
```

### 실행 결과 해석

배지는 `-5deg`와 `5deg` 사이를 네 회차 동안 오갑니다.
배율도 `1`과 `1.08` 사이에서 함께 변하므로 기울기만 바뀔 때보다 시선이 더 잘 모입니다.
네 번째 회차는 역방향이므로 마지막으로 계산된 시작 쪽 상태가 `forwards`에 의해 남습니다.
### 실무 연결

짧은 상태 배지, 신규 기능 표식, 처리 중 표시처럼 사용자가 알아야 할 변화를 부드럽게 알리는 데 활용할 수 있습니다.
무한 반복은 계속 시선을 빼앗을 수 있으므로 정보 전달 목적이라면 반복 횟수를 제한하는 편이 좋습니다.

---
## 직접 해보기

1. `@keyframes blink`를 만들었는데 요소에 `animation-name: pulse`를 지정하면 왜 실행되지 않는지 설명해 보세요.
2. 위 배지가 총 두 번 왕복한 뒤 멈추게 하려면 `iteration-count`를 몇으로 설정해야 하나요?
3. 1초 지연 중에도 첫 키프레임 상태를 보여 주고, 종료 후 마지막 상태도 유지하려면 어떤 `fill-mode`가 적절한가요?

<details>
<summary>정답 보기</summary>

1. 요소가 찾는 이름과 정의된 키프레임 이름이 다르기 때문입니다. 두 이름을 하나로 통일해야 합니다.
2. `alternate`의 한 회차는 한 방향이므로 왕복 한 번에 두 회차가 필요합니다. 두 번 왕복하려면 `4`로 설정합니다.
3. 재생 전에는 첫 키프레임을, 재생 후에는 마지막 키프레임을 적용하는 `both`가 적절합니다.

</details>
## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `transition` vs `animation` | 상태 변화 사이의 전환 vs 키프레임에 따른 독립적인 시간 재생 |
| `duration` vs `delay` | 한 회차가 걸리는 시간 vs 재생을 시작하기 전 기다리는 시간 |
| `iteration-count` vs `direction` | 몇 회 재생하는지 vs 각 회차를 어느 쪽으로 진행하는지 |
| `alternate` vs `reverse` | 회차마다 방향 교대 vs 모든 회차를 끝에서 시작으로 재생 |
| `forwards` vs `backwards` | 종료 뒤 마지막 상태 유지 vs 지연 중 시작 상태 적용 |
## 연결되는 개념

- 이전에 알면 좋은 개념: [전환과 hover 효과](02-transition-and-hover-effects.md)
- 다음에 이어지는 개념: [메뉴와 콘텐츠에 움직임 통합하기](04-interactive-page-animation.md)
- 전체 흐름 다시 보기: [움직이는 웹사이트와 반응형 웹 학습 지도](OVERVIEW.md)
- 함께 보면 좋은 키워드: `transform`, `timing-function`, `interactive-state`
## 셀프 체크

- [ ] `@keyframes`와 `animation-name`의 연결 관계를 설명할 수 있다.
- [ ] `duration`과 `delay`를 구분할 수 있다.
- [ ] 반복 횟수와 왕복 횟수의 관계를 계산할 수 있다.
- [ ] 네 가지 대표 방향 값을 비교할 수 있다.
- [ ] `fill-mode`가 재생 전후 스타일에 미치는 영향을 말할 수 있다.

### 복습 질문 및 답변

**Q1. 기본 — 애니메이션 이름만 지정했을 때 움직임이 보이지 않을 수 있는 이유는 무엇인가요?**

<details>
<summary>답</summary>

`animation-duration`의 기본값이 `0s`이기 때문입니다. 키프레임이 연결되어도 진행 시간이 없으면 중간 변화가 눈에 보이지 않습니다.

</details>

**Q2. 이해 확인 — `alternate`와 반복 횟수 `3`을 함께 사용하면 방향은 어떻게 진행되나요?**

<details>
<summary>답</summary>

첫 회차는 시작에서 끝, 두 번째는 끝에서 시작, 세 번째는 다시 시작에서 끝으로 진행합니다. 총 한 번 반의 왕복 흐름입니다.

</details>

**Q3. 응용 — 로딩 표시처럼 계속 움직여야 하지만 끝점에서 순간 이동하는 느낌은 줄이고 싶다면 어떤 값을 조합할 수 있나요?**

<details>
<summary>답</summary>

`animation-iteration-count: infinite`와 `animation-direction: alternate`를 조합할 수 있습니다. 끝점에 도달한 다음 반대 방향으로 이어서 움직이므로 시작점으로 갑자기 되감기는 느낌이 줄어듭니다.

</details>

## 한 줄 정리

> CSS 애니메이션은 키프레임으로 변화의 장면을 만들고, 시간·반복·방향·채움 규칙으로 그 장면의 재생 방식을 완성한다.
