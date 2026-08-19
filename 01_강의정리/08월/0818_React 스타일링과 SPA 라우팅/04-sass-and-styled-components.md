# Sass와 styled-components

두 도구 모두 CSS 작성 경험을 확장하지만, Sass는 빌드 전 변환에, styled-components는 컴포넌트 런타임 표현에 초점이 있습니다.

**키워드:** Sass, nesting, mixin, styled-components, tagged template

## 핵심 포인트

- Sass는 변수, 중첩, mixin으로 반복을 줄입니다.
- `&`는 현재 선택자를 가리킵니다.
- 중첩이 깊으면 생성된 선택자가 복잡해집니다.
- styled-components는 태그드 템플릿으로 스타일 컴포넌트를 만듭니다.
- `$` 접두 transient prop은 DOM 전달을 피하는 데 유용합니다.

## Sass로 규칙 재사용하기

```scss
$brand: #2563eb;

@mixin focus-ring($color: $brand) {
  outline: 3px solid rgba($color, .3);
  outline-offset: 2px;
}

.button {
  background: $brand;
  &:hover { filter: brightness(.9); }
  &:focus-visible { @include focus-ring; }
}
```

변수는 의미 있는 값을 공유하고 mixin은 여러 선언 묶음을 재사용합니다. `&`가 `.button`으로 치환되어 상태 선택자를 만듭니다. 중첩은 2~3단계 이내로 제한하면 구조를 추적하기 쉽습니다.

## styled-components로 변형 만들기

```jsx
const Button = styled.button`
  padding: 0.75rem 1rem;
  background: ${({ $tone }) => ($tone === 'danger' ? '#dc2626' : '#2563eb')};

  & + & { margin-left: 0.5rem; }
`;
```

목적은 버튼의 변형과 인접 간격을 컴포넌트에 묶는 것입니다. `$tone`을 받아 CSS 문자열을 계산하고, `& + &`는 같은 버튼이 연속될 때만 여백을 줍니다. 결과는 JSX에서 시맨틱한 컴포넌트를 재사용하는 형태입니다.

| Sass | styled-components |
|---|---|
| 빌드 시 일반 CSS로 변환 | JavaScript와 결합해 스타일 생성 |
| 기존 CSS 파일 흐름 유지 | props 기반 동적 표현이 자연스러움 |
| 변수·mixin 중심 | 컴포넌트 변형·테마 중심 |

## 연결해서 보기

- 선택 기준은 [React 스타일링 전략](01-react-styling-strategies.md)을 참고합니다.
- 레이아웃 규칙은 [Flexbox 반응형 레이아웃](03-flexbox-responsive-layout.md)에서 적용합니다.

## 직접 해보기

1. Sass mixin으로 공통 포커스 스타일을 만드세요.
2. `&`를 사용해 hover 상태를 작성하세요.
3. `$size` prop으로 버튼 padding을 바꾸세요.

<details>
<summary>답</summary>

`@mixin focus-ring { ... }`과 `@include focus-ring`을 사용합니다. hover는 `&:hover`, 동적 padding은 `${({ $size }) => $size === 'sm' ? '.5rem' : '.75rem'}`처럼 작성할 수 있습니다.

</details>

## 점검 목록

- [ ] Sass 중첩 깊이를 제한했다.
- [ ] mixin이 실제 반복을 줄이는지 확인했다.
- [ ] `&`가 치환될 선택자를 설명할 수 있다.
- [ ] 스타일 전용 prop이 DOM으로 전달되지 않는다.
- [ ] 동적 스타일의 필요성과 비용을 검토했다.

## 복습 질문 및 답변

### 1. Sass 변수와 CSS 변수의 큰 차이는?

<details>
<summary>답</summary>

Sass 변수는 빌드 시 치환되고 CSS 변수는 브라우저 런타임에도 상속과 변경이 가능합니다.

</details>

### 2. styled-components의 템플릿은 어떤 문법인가?

<details>
<summary>답</summary>

함수가 템플릿 문자열의 조각과 보간값을 받는 태그드 템플릿 문법입니다.

</details>

### 3. 깊은 중첩을 피해야 하는 이유는?

<details>
<summary>답</summary>

선택자 우선순위가 높아지고 구조 의존성이 커져 수정과 재사용이 어려워집니다.

</details>

## 정리

Sass는 정적 CSS의 생산성을, styled-components는 컴포넌트 상태와 스타일의 결합을 강화합니다.
