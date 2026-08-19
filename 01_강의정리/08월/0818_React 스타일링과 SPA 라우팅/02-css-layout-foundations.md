# CSS 레이아웃 기초

레이아웃이 예상과 다르게 보일 때는 대부분 박스의 실제 크기와 배치 기준을 잘못 이해한 경우입니다.

**키워드:** box model, box-sizing, position, containing block, CSS 단위

## 핵심 포인트

- 요소 크기는 content, padding, border, margin의 합으로 결정됩니다.
- `border-box`는 선언한 너비 안에 padding과 border를 포함합니다.
- `position`은 요소의 배치 흐름과 기준점을 바꿉니다.
- `%`, `rem`, `vw`는 서로 다른 기준에 상대적인 단위입니다.
- 고정값과 상대값을 목적에 맞게 섞어야 안정적입니다.

## 박스 모델과 크기 계산

```css
*, *::before, *::after { box-sizing: border-box; }

.panel {
  width: min(100%, 48rem);
  padding: 1.5rem;
  border: 1px solid #cbd5e1;
  margin-inline: auto;
}
```

목적은 화면보다 넓어지지 않는 패널을 만드는 것입니다. `border-box`를 적용한 뒤 최대 너비를 제한하고 자동 바깥 여백으로 가운데 정렬합니다. 결과는 padding을 추가해도 전체 너비가 예측 가능합니다.

## position의 기준

```css
.card { position: relative; }
.badge { position: absolute; top: 0.75rem; right: 0.75rem; }
.header { position: sticky; top: 0; }
```

`relative`는 원래 흐름을 유지하면서 자식 absolute의 기준을 만듭니다. `absolute`는 일반 흐름에서 빠지고 가장 가까운 위치 지정 조상을 기준으로 합니다. `fixed`는 뷰포트, `sticky`는 스크롤 임계점 전후의 흐름을 기준으로 동작합니다.

## 단위 선택

- `px`: 테두리처럼 작은 고정값
- `rem`: 글꼴과 간격처럼 루트 글꼴에 비례할 값
- `%`: 부모 크기에 비례할 값
- `vw`, `vh`: 뷰포트에 비례할 값
- `clamp()`: 최소·선호·최대 범위를 가진 반응형 값

```css
h1 { font-size: clamp(1.75rem, 4vw, 3rem); }
```

| `content-box` | `border-box` |
|---|---|
| width가 콘텐츠 너비만 의미 | width가 테두리까지 포함 |
| padding 추가 시 실제 크기 증가 | 선언한 크기 안에서 공간 배분 |
| 계산이 쉽게 어긋남 | 레이아웃 예측이 쉬움 |

## 연결해서 보기

- 한 축 정렬은 [Flexbox 반응형 레이아웃](03-flexbox-responsive-layout.md)을 참고합니다.
- 스타일 관리 전략은 [React 스타일링 전략](01-react-styling-strategies.md)에서 비교합니다.

## 직접 해보기

1. 너비 320px 카드에 padding 24px을 주고 두 box-sizing의 실제 너비를 비교하세요.
2. 카드 오른쪽 위에 absolute 배지를 배치하세요.
3. 제목 크기를 `clamp()`로 반응형 처리하세요.

<details>
<summary>답</summary>

`content-box`의 실제 너비는 테두리를 제외해도 368px, `border-box`는 320px입니다. 카드에 `position: relative`, 배지에 `position: absolute; top: 0; right: 0`을 사용합니다.

</details>

## 점검 목록

- [ ] 실제 박스 너비를 계산할 수 있다.
- [ ] 전역 `border-box`를 적용했다.
- [ ] absolute의 기준 조상을 확인했다.
- [ ] 상대 단위의 기준을 설명할 수 있다.
- [ ] 콘텐츠가 화면 밖으로 넘치는지 점검했다.

## 복습 질문 및 답변

### 1. margin은 width 계산에 포함되는가?

<details>
<summary>답</summary>

아니요. margin은 요소 바깥의 간격이며 `box-sizing` 계산 대상이 아닙니다.

</details>

### 2. sticky가 동작하지 않을 때 무엇을 확인하는가?

<details>
<summary>답</summary>

`top` 같은 임계값과 조상 요소의 overflow 및 스크롤 컨테이너를 확인합니다.

</details>

### 3. rem은 무엇을 기준으로 하는가?

<details>
<summary>답</summary>

루트 `html` 요소의 글꼴 크기를 기준으로 합니다.

</details>

## 정리

박스 크기, 위치 기준, 단위 기준을 명확히 하면 레이아웃 오류를 계산 가능한 문제로 바꿀 수 있습니다.
