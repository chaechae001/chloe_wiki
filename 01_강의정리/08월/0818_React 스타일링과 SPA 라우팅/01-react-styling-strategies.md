# React 스타일링 전략

컴포넌트가 늘어날수록 스타일의 핵심은 ‘예쁘게 만들기’보다 충돌을 막고 변경 범위를 예측하는 일이 됩니다.

**키워드:** CSS import, CSS Module, CSS-in-JS, 스코프, 동적 스타일

## 핵심 포인트

- 전역 CSS는 시작이 빠르지만 이름 충돌과 영향 범위 추적이 어렵습니다.
- CSS Module은 클래스 이름을 파일 단위로 격리합니다.
- CSS-in-JS는 JavaScript 값으로 동적 스타일을 표현하기 쉽습니다.
- 공통 토큰과 컴포넌트 스타일을 분리하면 일관성이 좋아집니다.
- 팀 규모, 런타임 비용, 디버깅 경험을 함께 보고 방식을 선택합니다.

## 세 가지 접근법

### 전역 CSS

```jsx
import './global.css';

export default function App() {
  return <button className="primary-button">저장</button>;
}
```

진입점에서 한 번 불러오며 리셋, 글꼴, 색상 변수처럼 앱 전체 규칙에 적합합니다. 모든 컴포넌트가 같은 네임스페이스를 공유하므로 클래스 이름 규칙이 필요합니다.

### CSS Module

```css
/* Button.module.css */
.button { padding: 0.75rem 1rem; }
.danger { background: #d33; color: white; }
```

```jsx
import styles from './Button.module.css';

export function Button({ danger, children }) {
  const className = `${styles.button} ${danger ? styles.danger : ''}`;
  return <button className={className}>{children}</button>;
}
```

빌드 도구가 클래스 이름을 고유하게 바꿉니다. 목적은 충돌 방지이고, 흐름은 파일 import → 매핑 객체 참조 → 변환된 클래스 적용입니다. 결과적으로 같은 `.button` 이름을 다른 파일에서도 안전하게 쓸 수 있습니다.

### CSS-in-JS

```jsx
const SaveButton = styled.button`
  background: ${({ $active }) => ($active ? '#2563eb' : '#94a3b8')};
  color: white;
`;

<SaveButton $active={isValid}>저장</SaveButton>
```

props를 스타일 결정에 직접 연결할 수 있습니다. 실무에서는 상태에 따른 배지, 테마, 변형이 많은 디자인 시스템에 유용합니다. 라이브러리별 런타임 비용과 서버 렌더링 설정은 확인해야 합니다.

## 선택 기준

| CSS Module | CSS-in-JS |
|---|---|
| 정적 스타일과 낮은 런타임 비용에 유리 | props 기반 변형과 테마에 유리 |
| 브라우저 개발자 도구 흐름이 익숙함 | 컴포넌트와 스타일을 함께 관리 |
| 조건부 클래스 조합이 필요 | 라이브러리 의존성과 변환 비용이 있음 |

## 연결해서 보기

- 레이아웃 규칙은 [CSS 레이아웃 기초](02-css-layout-foundations.md)에서 다룹니다.
- 재사용 가능한 배치는 [Flexbox 반응형 레이아웃](03-flexbox-responsive-layout.md)으로 이어집니다.
- 전처리기와 CSS-in-JS 문법은 [Sass와 styled-components](04-sass-and-styled-components.md)를 참고합니다.

## 직접 해보기

1. 전역 `.card` 클래스를 CSS Module로 옮겨 보세요.
2. `variant` 값에 따라 버튼 색을 바꿔 보세요.
3. 색상과 간격을 CSS 변수로 분리해 보세요.

<details>
<summary>답</summary>

`Card.module.css`를 만들고 `styles.card`를 사용합니다. 버튼은 `variant`를 클래스 매핑이나 styled-components의 props 함수에 전달합니다. 공통 값은 `:root`의 `--color-primary`, `--space-md`처럼 선언할 수 있습니다.

</details>

## 점검 목록

- [ ] 전역 규칙과 지역 규칙을 구분했다.
- [ ] 클래스 이름 충돌 가능성을 확인했다.
- [ ] 동적 스타일이 정말 필요한지 판단했다.
- [ ] 색상과 간격 토큰을 재사용한다.
- [ ] 선택한 방식의 성능·디버깅 비용을 설명할 수 있다.

## 복습 질문 및 답변

### 1. CSS Module이 충돌을 줄이는 이유는?

<details>
<summary>답</summary>

빌드 시 클래스 이름을 파일별 고유 이름으로 변환하기 때문입니다.

</details>

### 2. CSS-in-JS가 특히 편한 상황은?

<details>
<summary>답</summary>

props, 상태, 테마에 따라 스타일 변형이 많을 때입니다.

</details>

### 3. 전역 CSS가 여전히 필요한 이유는?

<details>
<summary>답</summary>

리셋, 글꼴, 루트 변수처럼 앱 전체가 공유할 기반 규칙이 있기 때문입니다.

</details>

## 정리

스타일링 방식은 하나로 통일하기보다 전역 기반, 지역 스코프, 동적 변형의 책임을 나누어 선택합니다.
