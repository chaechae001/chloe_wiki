# 컴포넌트와 children, 렌더링

컴포넌트는 UI를 책임 단위로 나누고, Props와 children을 통해 같은 구조를 다양한 내용으로 재사용하게 합니다.

## 핵심 키워드

`component` · `function component` · `children` · `render phase` · `commit phase`

## 핵심 요약

- 함수 컴포넌트는 Props를 받아 JSX를 반환합니다.
- 컴포넌트 이름은 기본 태그와 구분되도록 대문자로 시작합니다.
- children은 여는 태그와 닫는 태그 사이의 내용을 전달합니다.
- 렌더 단계는 결과를 계산하고 커밋 단계는 실제 DOM에 반영합니다.

## 1. 함수 컴포넌트와 재사용

```jsx
function Tag({ label }) {
  return <span className="tag">{label}</span>;
}

function App() {
  return <><Tag label="React" /><Tag label="JSX" /></>;
}
```

같은 `Tag` 구조에 다른 값을 전달합니다. 현대 React에서는 함수 컴포넌트와 Hooks를 중심으로 작성하는 경우가 많습니다.

## 2. children으로 내용 조합하기

```jsx
function Panel({ title, children }) {
  return (
    <section className="panel">
      <h2>{title}</h2>
      <div>{children}</div>
    </section>
  );
}

function App() {
  return <Panel title="공지"><p>오늘의 학습 내용을 확인하세요.</p></Panel>;
}
```

Panel은 내부 내용의 종류를 몰라도 공통 레이아웃을 제공합니다. children은 문자열, 요소, 컴포넌트 배열 등 렌더 가능한 값일 수 있습니다.

## 3. 렌더와 커밋

렌더 단계에서 React는 Props와 State를 바탕으로 다음 UI 결과를 계산합니다. 커밋 단계에서는 계산 결과에 필요한 실제 DOM 변경을 반영합니다. 렌더 함수 안에서는 외부 시스템 변경 같은 부수 효과를 피해야 같은 입력에 같은 결과를 기대할 수 있습니다.

## 대표 코드: 조립 가능한 카드

### 목적

공통 카드 구조에 서로 다른 본문과 동작을 전달합니다.

```jsx
function Card({ title, children, footer }) {
  return (
    <article>
      <h3>{title}</h3>
      <div>{children}</div>
      <footer>{footer}</footer>
    </article>
  );
}

function CourseCard() {
  return (
    <Card title="JSX 기초" footer={<button>학습하기</button>}>
      <p>컴포넌트 작성 규칙을 익힙니다.</p>
    </Card>
  );
}
```

### 코드 흐름과 결과

1. Card가 공통 구조를 담당합니다.
2. title은 일반 Props로 전달합니다.
3. 본문은 children으로 전달합니다.
4. footer에는 JSX 요소 자체를 Props로 전달해 조합합니다.

### 실무 연결

모달, 레이아웃, 카드, 탭처럼 틀은 같지만 내부 내용이 달라지는 UI를 만들 때 유용합니다.

## 직접 해보기

1. 컴포넌트 이름을 대문자로 시작하는 이유를 설명하세요.
2. children을 감싸는 `Layout` 컴포넌트를 작성하세요.
3. 렌더 중 네트워크 요청을 직접 실행하면 안 되는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 소문자 이름은 기본 DOM 태그로 해석되므로 사용자 정의 컴포넌트와 구분하기 위해서입니다.
2. `function Layout({ children }) { return <main>{children}</main>; }`처럼 작성합니다.
3. 렌더는 여러 번 실행될 수 있어 요청이 중복되고 결과 계산에 부수 효과가 섞일 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 컴포넌트 함수 vs 컴포넌트 사용 | 함수는 정의이고 `<Component />`는 React에 렌더링을 요청하는 요소입니다. |
| children vs 일반 Props | children은 태그 사이 내용을 담고 일반 Props는 이름으로 전달합니다. |
| 렌더 vs 커밋 | 렌더는 UI 계산, 커밋은 실제 DOM 반영 단계입니다. |

## 연결되는 개념

- JSX 구조는 [JSX 문법과 기본 규칙](01-jsx-syntax-and-rules.md)에서 설명합니다.
- 외부 입력은 [Props와 단방향 데이터 흐름](04-props-and-data-flow.md)에서 이어집니다.
- 내부 변화는 [State와 불변 갱신](05-state-and-immutable-updates.md)에서 다룹니다.

## 셀프 체크

- [ ] 함수 컴포넌트를 작성할 수 있다.
- [ ] children으로 UI를 조합할 수 있다.
- [ ] 렌더와 커밋 단계를 구분할 수 있다.

## 복습 질문 및 답변

### Q1. 함수 컴포넌트의 반환값은?

<details>
<summary>답</summary>

React가 렌더링할 JSX 또는 렌더 가능한 값입니다.

</details>

### Q2. children은 반드시 한 요소인가?

<details>
<summary>답</summary>

아닙니다. 태그 사이에 전달된 여러 렌더 가능한 값을 포함할 수 있습니다.

</details>

### Q3. 같은 컴포넌트를 재사용하는 핵심 방법은?

<details>
<summary>답</summary>

공통 구조를 컴포넌트에 두고 달라지는 데이터와 내용을 Props와 children으로 전달합니다.

</details>

## 한 줄 정리

> 컴포넌트는 Props와 children으로 UI를 조합하고 렌더 결과를 실제 DOM 변경과 분리합니다.
