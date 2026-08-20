# Redux와 Redux Toolkit

> Redux는 상태 자체보다 상태가 바뀌는 경로를 표준화해 규모가 큰 앱의 협업과 추적을 돕습니다.

`Store` · `Action` · `Reducer` · `Selector` · `Redux Toolkit`

## 핵심요약

- Redux Store는 앱의 공유 상태를 한 데이터 트리로 관리합니다.
- 컴포넌트는 Action을 dispatch하고 Reducer가 다음 State를 만듭니다.
- Selector는 Store에서 필요한 조각이나 계산 결과를 읽습니다.
- Middleware는 Action이 Reducer에 도달하기 전의 확장 지점입니다.
- Redux Toolkit은 권장 설정과 반복 코드를 줄이는 API를 제공합니다.

## 1. Redux 데이터 흐름

```text
UI event → dispatch(action) → middleware → reducer → store → selector → UI
```

Action은 보통 `type`과 `payload`를 가진 객체입니다. Reducer는 `(state, action) => nextState` 규칙을 따릅니다. Store는 새 상태를 보관하고 구독자에게 변경을 알립니다. Selector는 원본 상태를 바꾸지 않고 필요한 값을 선택합니다.

## 2. createSlice로 상태 단위 구성하기

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    itemAdded(state, action) {
      state.items.push(action.payload);
    },
    itemRemoved(state, action) {
      state.items = state.items.filter((item) => item.id !== action.payload);
    },
  },
});

export const store = configureStore({
  reducer: { cart: cartSlice.reducer },
});
export const { itemAdded, itemRemoved } = cartSlice.actions;
```

코드가 상태를 직접 바꾸는 것처럼 보이지만 Toolkit 내부의 Immer가 안전한 불변 업데이트로 변환합니다. `createSlice`는 Action creator와 Reducer를 같은 기능 단위에 모읍니다.

## 3. React에서 연결하기

```javascript
import { Provider, useDispatch, useSelector } from 'react-redux';

function CartCount() {
  const count = useSelector((state) => state.cart.items.length);
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(itemAdded({ id: crypto.randomUUID() }))}>{count}</button>;
}

root.render(<Provider store={store}><CartCount /></Provider>);
```

### 코드 목적

Store를 React 트리에 제공하고 컴포넌트가 선택한 상태를 읽고 Action을 보냅니다.

### 코드 흐름과 결과 해석

클릭이 Action을 dispatch하면 slice reducer가 cart state를 갱신합니다. Selector 결과인 개수가 바뀌어 해당 컴포넌트가 새 값으로 렌더링됩니다.

### 실무 연결

여러 화면이 공유하는 편집 상태, 권한, 복잡한 업무 흐름처럼 변경 기록과 개발 도구가 중요한 앱에 적합합니다. 단순한 로컬 입력까지 Redux에 넣으면 복잡도만 늘 수 있습니다.

## 4. Selector와 Middleware

Memoized selector는 입력 상태가 같을 때 계산 결과를 재사용할 수 있습니다. Middleware는 로깅, 비동기 함수 처리, 분석 이벤트처럼 Reducer 바깥에서 필요한 작업을 맡습니다.

## 직접 해보기

1. 장바구니 전체 개수를 계산하는 selector를 작성하세요.
2. `createSlice`가 줄이는 반복 코드를 설명하세요.
3. 로컬 input 값을 Redux에 둘지 판단하고 근거를 말하세요.

<details>
<summary>답</summary>

1. `(state) => state.cart.items.length`처럼 작성할 수 있습니다.
2. Action type, Action creator, switch reducer 선언을 기능 단위로 함께 생성합니다.
3. 다른 화면과 공유하거나 복원·추적할 필요가 없다면 로컬 state가 더 단순합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Store vs slice | 앱 상태 전체 보관소 vs 기능별 상태와 변경 규칙 묶음 |
| Reducer vs selector | 다음 상태 생성 vs 현재 상태 읽기·계산 |
| Middleware vs enhancer | dispatch 경로 확장 vs Store 생성 기능 확장 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [Flux와 useReducer](05-flux-and-usereducer.md)
- 다음에 이어지는 개념: [Redux 비동기 상태 관리](07-redux-async-state.md)

## 셀프 체크

- [ ] Redux 데이터 흐름을 순서대로 설명한다.
- [ ] Store, Action, Reducer를 구분한다.
- [ ] createSlice의 결과물을 이해한다.
- [ ] Provider, useSelector, useDispatch를 연결한다.
- [ ] Redux가 필요한 상태인지 판단할 수 있다.

### 복습 질문 및 답변

**Q1. Selector가 상태를 변경해도 되는가?**

<details>
<summary>답</summary>

안 됩니다. Selector는 상태를 읽거나 계산한 값을 반환하는 순수한 읽기 함수여야 합니다.

</details>

**Q2. Toolkit reducer에서 push 문법을 쓸 수 있는 이유는?**

<details>
<summary>답</summary>

내부 Immer가 변경 문법을 바탕으로 불변성을 지킨 새 상태를 생성하기 때문입니다.

</details>

**Q3. Redux의 single source of truth가 모든 값을 하나의 객체에 직접 넣으라는 뜻인가?**

<details>
<summary>답</summary>

공유 Redux 상태가 하나의 Store 트리에서 관리된다는 뜻이며, 모든 로컬 UI 상태까지 전역화하라는 의미는 아닙니다.

</details>

## 한 줄 정리

> Redux Toolkit은 Action에서 Store 갱신까지의 표준 흐름을 유지하면서 Redux 구현의 반복을 줄여 줍니다.
