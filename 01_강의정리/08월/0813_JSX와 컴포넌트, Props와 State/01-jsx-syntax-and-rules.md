# JSX 문법과 기본 규칙

JSX는 JavaScript 안에서 UI 구조를 HTML과 비슷하게 표현하고, 데이터와 화면을 한 흐름으로 연결하는 문법입니다.

## 핵심 키워드

`JSX` · `Babel` · `expression` · `fragment` · `self-closing tag`

## 핵심 요약

- JSX는 브라우저가 직접 실행하는 HTML이 아니라 JavaScript로 변환되는 문법입니다.
- 중괄호 안에는 값을 만드는 JavaScript 표현식을 넣습니다.
- 컴포넌트는 하나의 최상위 요소를 반환해야 합니다.
- 모든 태그는 닫혀 있어야 하며 내용이 없으면 축약형을 사용할 수 있습니다.

## 1. JSX가 필요한 이유

UI 구조와 그 구조에 들어갈 데이터를 가까이 작성하면 화면이 어떤 값에서 만들어지는지 읽기 쉽습니다. JSX는 변환 도구를 거쳐 React 요소를 만드는 JavaScript 호출로 바뀝니다.

```jsx
const user = { name: "새싹", point: 120 };

function Profile() {
  return (
    <section>
      <h1>{user.name}</h1>
      <p>점수: {user.point + 10}</p>
    </section>
  );
}
```

중괄호에는 변수, 연산, 함수 호출처럼 결과값을 만드는 표현식을 넣을 수 있습니다. `if`문이나 `for`문처럼 그 자체로 값이 되지 않는 문장은 바로 넣을 수 없습니다.

## 2. 태그와 최상위 요소 규칙

JSX 태그는 반드시 닫아야 합니다. 반환할 형제 요소가 여러 개라면 의미 있는 컨테이너나 Fragment로 감쌉니다.

```jsx
function Summary() {
  return (
    <>
      <h2>오늘의 학습</h2>
      <img src="/study.png" alt="학습 기록" />
    </>
  );
}
```

Fragment는 불필요한 DOM 요소를 추가하지 않으면서 하나의 JSX 결과로 묶습니다.

## 대표 코드: 조건에 따른 메시지

### 목적

JavaScript 표현식을 이용해 상태에 맞는 UI 문구를 선택합니다.

```jsx
function ProgressMessage({ completed, total }) {
  const finished = completed === total;

  return (
    <p>
      {finished ? "모든 학습을 완료했습니다." : `${completed}/${total} 진행 중`}
    </p>
  );
}
```

### 코드 흐름과 결과

1. Props에서 완료 수와 전체 수를 받습니다.
2. 두 값이 같은지 계산합니다.
3. 삼항 연산자가 화면에 표시할 문자열을 선택합니다.
4. 데이터가 바뀌면 같은 규칙으로 다른 JSX가 만들어집니다.

### 실무 연결

로그인 여부, 로딩 상태, 권한, 데이터 존재 여부에 따른 조건부 UI를 만들 때 사용합니다.

## 직접 해보기

1. JSX 중괄호에 `if`문을 바로 넣을 수 없는 이유를 설명하세요.
2. 제목과 설명을 Fragment로 묶어 반환하세요.
3. 이미지 태그를 닫지 않았을 때 JSX에서 오류가 나는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 중괄호는 값으로 평가되는 표현식 위치이며 `if`는 문장이기 때문입니다.
2. `return <><h1>제목</h1><p>설명</p></>;`처럼 작성할 수 있습니다.
3. JSX는 XML과 유사한 엄격한 태그 구조로 변환되므로 모든 태그를 닫아야 합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| JSX vs HTML | JSX는 JavaScript로 변환되는 UI 문법이며 속성명과 태그 규칙이 일부 다릅니다. |
| 표현식 vs 문장 | 표현식은 값을 만들지만 `if`, `for` 같은 문장은 값을 직접 반환하지 않습니다. |
| Fragment vs div | Fragment는 실제 DOM 노드를 추가하지 않고 형제 요소를 묶습니다. |

## 연결되는 개념

- JSX 속성은 [스타일과 React DOM 속성](02-styles-and-dom-attributes.md)에서 이어집니다.
- JSX를 반환하는 단위는 [컴포넌트와 children](03-components-children-and-rendering.md)에서 다룹니다.
- 용어는 [용어집](GLOSSARY.md)을 확인하세요.

## 셀프 체크

- [ ] JSX의 변환 과정을 설명할 수 있다.
- [ ] 표현식을 중괄호로 삽입할 수 있다.
- [ ] 태그와 최상위 요소 규칙을 지킬 수 있다.

## 복습 질문 및 답변

### Q1. JSX는 브라우저가 그대로 이해하는가?

<details>
<summary>답</summary>

일반적으로 변환 도구가 브라우저가 실행할 JavaScript로 바꿉니다.

</details>

### Q2. 여러 최상위 요소를 반환하려면?

<details>
<summary>답</summary>

하나의 부모 요소 또는 Fragment로 감쌉니다.

</details>

### Q3. 중괄호 안에서 객체를 그대로 출력할 수 있는가?

<details>
<summary>답</summary>

일반 객체는 React 자식으로 직접 렌더링할 수 없으므로 필요한 프로퍼티나 문자열 변환 결과를 사용해야 합니다.

</details>

## 한 줄 정리

> JSX는 데이터 표현식과 엄격한 태그 구조를 이용해 UI를 JavaScript 안에서 선언하는 문법입니다.
