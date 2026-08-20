# Flux와 useReducer

> 상태 변경을 ‘무슨 일이 일어났는가’라는 Action으로 표현하면 복잡한 UI도 흐름을 추적하기 쉬워집니다.

`Flux` · `단방향 데이터 흐름` · `Action` · `Reducer` · `Dispatch`

## 핵심요약

- Flux는 Action에서 Store와 View로 이어지는 단방향 흐름을 강조합니다.
- Action은 발생한 사건을 설명하고 Reducer는 다음 상태를 계산합니다.
- `useReducer`는 관련된 여러 상태 변경 규칙을 한곳에 모읍니다.
- Reducer는 같은 입력에 같은 출력을 내는 순수 함수로 작성합니다.
- 새 상태를 만들 때 기존 상태를 직접 변경하지 않습니다.

## 1. 단방향 데이터 흐름

사용자 입력이 Action을 만들고, 상태 변경 로직이 새 State를 계산하며, View가 그 State를 렌더링합니다. 다시 발생한 사용자 입력은 새 Action이 됩니다. 업데이트 경로가 한 방향이므로 여러 화면이 같은 데이터를 사용할 때도 원인을 추적하기 쉽습니다.

```text
View → Action → Reducer → State → View
```

## 2. useReducer로 변경 규칙 모으기

```javascript
const initialState = { items: [], filter: 'all', nextId: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'item/added':
      return {
        ...state,
        items: [...state.items, { id: state.nextId, title: action.payload, done: false }],
        nextId: state.nextId + 1,
      };
    case 'item/toggled':
      return {
        ...state,
        items: state.items.map((item) =>
          item.id === action.payload ? { ...item, done: !item.done } : item
        ),
      };
    case 'filter/changed':
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}
```

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: 'item/added', payload: '문서 검토' });
```

### 코드 목적

추가, 완료 전환, 필터 변경을 하나의 상태 머신처럼 관리합니다.

### 코드 흐름과 결과 해석

컴포넌트가 Action을 dispatch하면 React가 현재 state와 action으로 reducer를 호출합니다. Reducer는 기존 객체를 바꾸지 않고 새 객체를 반환하고, React는 새 상태로 UI를 렌더링합니다.

### 실무 연결

폼 단계, 장바구니, 편집기처럼 하나의 상태에 여러 사건이 작용할 때 `setState` 콜백을 흩어 놓는 것보다 변경 규칙을 테스트하고 추적하기 쉽습니다.

## 3. Context와 조합하기

Provider 내부에서 `useReducer`를 실행하고 state와 dispatch 또는 의미 있는 Action 함수를 Context로 제공하면 깊은 자식이 공통 상태를 사용할 수 있습니다. 읽기 전용 Context와 dispatch Context를 나누면 변경되지 않은 소비자의 렌더링 범위를 줄이는 데 도움이 됩니다.

## 직접 해보기

1. 항목 삭제 Action과 reducer 분기를 추가하세요.
2. reducer 안에서 `state.items.push()`를 직접 사용하는 문제를 설명하세요.
3. Context에 state와 dispatch를 나눠 제공하는 이유를 말하세요.

<details>
<summary>답</summary>

1. `item/deleted` Action을 받고 `filter`로 해당 ID를 제외한 새 배열을 반환합니다.
2. 기존 상태를 변경하면 이전·다음 상태 비교와 변경 추적이 어려워집니다.
3. dispatch만 쓰는 컴포넌트가 state 변경에 불필요하게 반응하는 일을 줄일 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Action vs dispatch | 사건을 나타내는 값 vs 그 사건을 상태 시스템에 전달하는 함수 |
| Reducer vs event handler | 다음 상태 계산만 담당 vs UI 이벤트와 부수 효과 처리 가능 |
| `useState` vs `useReducer` | 단순 독립 상태에 편리 vs 관련 변경 규칙이 많을 때 유리 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [React 상태 관리와 Context](04-react-state-and-context.md)
- 다음에 이어지는 개념: [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md)

## 셀프 체크

- [ ] 단방향 데이터 흐름을 설명한다.
- [ ] Action과 dispatch를 구분한다.
- [ ] 순수 reducer를 작성한다.
- [ ] 불변성을 지키며 배열을 갱신한다.
- [ ] useReducer를 도입할 기준을 말할 수 있다.

### 복습 질문 및 답변

**Q1. reducer에서 네트워크 요청을 직접 하지 않는 이유는?**

<details>
<summary>답</summary>

Reducer를 순수하고 예측 가능하게 유지해 같은 state와 action에서 같은 결과를 만들기 위해서입니다.

</details>

**Q2. 알 수 없는 Action에서 무엇을 반환해야 하는가?**

<details>
<summary>답</summary>

일반적으로 변경하지 않은 현재 state를 반환합니다.

</details>

**Q3. Flux의 단방향 흐름이 디버깅에 유리한 이유는?**

<details>
<summary>답</summary>

상태 변경이 Action과 Reducer를 거치는 일정한 경로를 따르므로 원인과 결과를 역추적하기 쉽습니다.

</details>

## 한 줄 정리

> Flux와 useReducer는 상태 변경을 명시적인 Action과 순수한 다음 상태 계산으로 구조화합니다.
