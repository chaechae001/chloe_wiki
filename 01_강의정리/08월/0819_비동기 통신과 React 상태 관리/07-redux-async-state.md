# Redux 비동기 상태 관리

> 비동기 데이터는 값 하나가 아니라 대기, 성공, 실패라는 시간의 상태를 함께 관리해야 합니다.

`createAsyncThunk` · `pending` · `fulfilled` · `rejected` · `extraReducers`

## 핵심요약

- Reducer 밖에서 비동기 작업을 실행하고 결과를 Action으로 전달합니다.
- `createAsyncThunk`는 Promise 생명주기에 맞춘 세 Action을 생성합니다.
- `extraReducers`에서 loading, data, error를 각각 갱신합니다.
- 컴포넌트는 thunk를 dispatch하고 Store 상태로 UI를 렌더링합니다.
- 여러 thunk의 실패를 일반 Promise와 동일하다고 가정하지 말고 결과 해제 방식을 확인합니다.

## 1. 비동기 thunk 정의하기

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async (_, { rejectWithValue }) => {
    const response = await fetch('/api/products');
    if (!response.ok) return rejectWithValue(`HTTP ${response.status}`);
    return response.json();
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState: { items: [], status: 'idle', error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.status = 'loading';
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload ?? action.error.message;
      });
  },
});
```

### 코드 목적

상품 조회의 요청 상태와 결과, 오류를 하나의 slice에서 일관되게 관리합니다.

### 코드 흐름과 결과 해석

Thunk dispatch 직후 `pending`, 성공 시 반환 데이터가 담긴 `fulfilled`, 실패 시 `rejected` Action이 발생합니다. UI는 `status`에 따라 로딩, 목록, 오류 화면을 선택할 수 있습니다.

### 실무 연결

여러 화면이 같은 요청 결과를 공유하거나 요청 이력과 상태 변경을 개발 도구에서 추적해야 할 때 유용합니다. 단순 서버 캐싱 요구가 중심이라면 전용 데이터 패칭 도구도 비교할 수 있습니다.

## 2. 컴포넌트에서 결과 다루기

```javascript
function ReloadButton() {
  const dispatch = useDispatch();

  async function handleClick() {
    try {
      const items = await dispatch(fetchProducts()).unwrap();
      console.log(`${items.length}개 로드`);
    } catch (error) {
      console.error('조회 실패', error);
    }
  }

  return <button onClick={handleClick}>새로고침</button>;
}
```

Dispatch가 반환하는 결과 Action을 `.unwrap()`하면 fulfilled payload는 값으로 받고 rejected 결과는 throw되어 일반적인 `try/catch` 흐름으로 다룰 수 있습니다.

## 3. 중복 요청과 오래된 응답

검색어가 빠르게 바뀌면 이전 요청이 늦게 도착해 최신 결과를 덮을 수 있습니다. request ID 확인, AbortController, 조건부 실행 또는 캐시 정책으로 경쟁 상태를 관리합니다.

## 직접 해보기

1. `status`가 loading일 때 버튼을 비활성화하세요.
2. 저장 thunk의 성공 payload를 목록 끝에 추가하세요.
3. 빠른 검색에서 이전 응답이 최신 결과를 덮는 문제의 해결책을 제안하세요.

<details>
<summary>답</summary>

1. Selector로 status를 읽어 `disabled={status === 'loading'}`을 사용합니다.
2. fulfilled reducer에서 `state.items.push(action.payload)`로 반영할 수 있습니다.
3. 이전 요청을 취소하거나 request ID를 비교해 최신 요청의 결과만 반영합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `reducers` vs `extraReducers` | slice가 직접 만든 Action 처리 vs 외부·thunk Action 처리 |
| `action.payload` vs `action.error` | 성공·사용자 정의 실패 값 vs 기본 오류 정보 |
| dispatch 결과 vs `unwrap()` 결과 | 완료 Action 객체 vs payload 또는 throw된 오류 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md)
- 기초 비동기 흐름: [async/await와 Promise 조합](02-async-await-and-promise-combinators.md)

## 셀프 체크

- [ ] 비동기 상태 세 단계를 UI와 연결한다.
- [ ] createAsyncThunk의 payload creator를 설명한다.
- [ ] extraReducers에 세 상태를 작성한다.
- [ ] unwrap의 역할을 이해한다.
- [ ] 중복 요청과 경쟁 상태를 고려한다.

### 복습 질문 및 답변

**Q1. 비동기 요청을 reducer 안에서 직접 실행하면 안 되는 이유는?**

<details>
<summary>답</summary>

Reducer가 순수한 다음 상태 계산이라는 규칙을 잃어 실행 결과를 예측하고 테스트하기 어려워집니다.

</details>

**Q2. pending에서 이전 error를 지우는 이유는?**

<details>
<summary>답</summary>

새 요청이 시작됐는데 이전 실패 메시지가 계속 표시되는 상태를 피하기 위해서입니다.

</details>

**Q3. 모든 API 데이터를 Redux에 넣어야 하는가?**

<details>
<summary>답</summary>

아닙니다. 공유 범위, 캐싱, 동기화, 추적 요구를 보고 로컬 상태나 전용 서버 상태 도구와 비교해야 합니다.

</details>

## 한 줄 정리

> Redux 비동기 상태 관리는 요청의 생명주기를 Action으로 바꾸고 Store에서 로딩·성공·실패를 명시적으로 관리합니다.
