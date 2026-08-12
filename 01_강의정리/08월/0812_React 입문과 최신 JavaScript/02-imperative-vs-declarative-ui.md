# 명령형 DOM과 선언형 UI

직접 DOM을 수정하는 방식은 변경 절차를 쓰고, React는 현재 상태에서 보여야 할 결과를 선언합니다.

## 핵심 키워드

`imperative` · `declarative` · `DOM` · `rendering` · `state`

## 핵심 요약

- 명령형 코드는 요소 선택과 수정 순서를 직접 지시합니다.
- 선언형 코드는 데이터로부터 UI 결과를 표현합니다.
- 목록 변경이 많을수록 데이터와 DOM을 수동 동기화하기 어렵습니다.
- React는 상태 변화에 맞춰 UI 계산을 다시 수행합니다.

## 1. 두 방식의 차이

명령형 방식에서는 요소를 찾고, 만들고, 속성을 넣고, 부모에 추가하는 절차를 작성합니다. 선언형 방식에서는 배열을 JSX 목록으로 변환해 현재 결과를 표현합니다.

```javascript
const list = document.querySelector("#items");
const item = document.createElement("li");
item.textContent = "학습하기";
list.append(item);
```

```jsx
function ItemList({ items }) {
  return <ul>{items.map((item) => <li key={item.id}>{item.title}</li>)}</ul>;
}
```

둘 다 화면을 만들지만 첫 코드는 **어떻게 바꿀지**, 둘째 코드는 **무엇이 보여야 할지**에 초점을 둡니다.

## 2. 상태를 단일 기준으로 두기

화면 요소 자체와 별도 데이터가 동시에 진실의 원천이 되면 둘이 어긋날 수 있습니다. React에서는 상태를 기준으로 UI를 계산하여 데이터 흐름을 단순화합니다.

| 상황 | 직접 DOM 조작 | 상태 기반 렌더링 |
|---|---|---|
| 항목 추가 | 노드를 생성해 붙임 | 새 배열을 상태로 설정 |
| 항목 삭제 | 대상 노드를 찾아 제거 | 배열에서 항목을 제외 |
| 표시 갱신 | 여러 DOM 위치를 각각 수정 | 상태로 UI를 다시 계산 |

## 대표 코드: 목록 필터 표현

### 목적

완료 여부에 따라 목록을 선언적으로 계산합니다.

```jsx
function TaskList({ tasks, showDone }) {
  const visibleTasks = tasks.filter((task) => showDone || !task.done);

  return (
    <ul>
      {visibleTasks.map((task) => (
        <li key={task.id}>{task.title}</li>
      ))}
    </ul>
  );
}
```

### 코드 흐름과 결과

1. 원본 배열과 표시 조건을 입력받습니다.
2. `filter`로 현재 보여 줄 데이터만 계산합니다.
3. `map`으로 각 데이터를 JSX 요소로 바꿉니다.
4. 조건이 바뀌면 같은 규칙으로 UI가 다시 계산됩니다.

### 실무 연결

검색 결과, 장바구니, 알림 목록처럼 데이터가 자주 추가·삭제·필터링되는 화면에 선언형 접근이 특히 유용합니다.

## 직접 해보기

1. 명령형과 선언형의 초점을 비교하세요.
2. `tasks`에서 미완료 항목 수를 계산하세요.
3. DOM을 직접 수정하면서 별도 배열도 관리할 때 생기는 오류를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 명령형은 변경 절차, 선언형은 상태에 따른 결과를 기술합니다.
2. `tasks.filter((task) => !task.done).length`로 계산할 수 있습니다.
3. 한쪽만 변경되면 데이터와 화면이 서로 다른 상태를 나타낼 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 선언형 vs 자동화 | 선언형이어도 데이터 변환과 이벤트 규칙은 개발자가 작성합니다. |
| 원본 데이터 vs 파생 데이터 | 원본은 상태로 보관하고 필터 결과 같은 파생값은 계산할 수 있습니다. |
| 렌더링 vs DOM 전체 재생성 | React는 UI 결과를 비교해 필요한 변경을 실제 DOM에 반영합니다. |

## 연결되는 개념

- React의 목적은 [React의 역할과 학습 방향](01-react-overview.md)에서 설명합니다.
- 배열 변환은 [변수 선언과 배열 메서드](03-variables-and-array-methods.md)에서 이어집니다.
- 목록과 상태는 [컴포넌트와 State로 목록 설계](06-components-and-state.md)에서 다룹니다.

## 셀프 체크

- [ ] 명령형과 선언형 코드를 구분할 수 있다.
- [ ] 상태를 UI의 기준으로 삼는 이유를 설명할 수 있다.
- [ ] filter와 map으로 목록 UI를 표현할 수 있다.

## 복습 질문 및 답변

### Q1. 선언형 UI에서 개발자가 주로 기술하는 것은?

<details>
<summary>답</summary>

현재 데이터와 상태가 주어졌을 때 어떤 UI가 보여야 하는지입니다.

</details>

### Q2. 목록 항목에 안정적인 key가 필요한 이유는?

<details>
<summary>답</summary>

React가 이전 항목과 다음 항목의 정체성을 비교해 변경된 요소를 올바르게 추적하도록 돕기 위해서입니다.

</details>

### Q3. 필터 결과를 항상 별도 상태로 저장해야 하는가?

<details>
<summary>답</summary>

아닙니다. 원본 상태와 조건에서 계산 가능한 파생 데이터는 렌더링 중 계산해 중복 상태를 피할 수 있습니다.

</details>

## 한 줄 정리

> 선언형 UI는 DOM 변경 절차보다 현재 데이터가 표현해야 할 화면 결과에 집중합니다.
