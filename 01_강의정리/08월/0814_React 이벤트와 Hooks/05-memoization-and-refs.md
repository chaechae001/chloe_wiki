# 메모이제이션과 ref

React 최적화는 Hook을 많이 쓰는 일이 아니라, 어떤 값을 다시 계산할 필요가 없는지 측정하고 경계를 분명히 하는 일입니다. `useMemo`, `useCallback`, `useRef`는 서로 다른 대상을 보존합니다.

`useMemo` · `useCallback` · `useRef` · `memoization` · `DOM reference`

## 핵심요약

- `useMemo`는 의존성이 같을 때 계산 결과를 재사용합니다.
- `useCallback`은 의존성이 같을 때 함수 참조를 재사용합니다.
- `useRef`는 렌더링 사이에 값을 보존하지만 `.current` 변경만으로 재렌더링하지 않습니다.
- 메모이제이션은 정확성을 위한 도구가 아니라 불필요한 작업을 줄이는 최적화 도구입니다.
- 모든 계산과 함수를 무조건 메모이제이션하면 복잡성만 늘 수 있으므로 실제 필요성을 먼저 확인합니다.

## 1. 계산 결과를 보존하는 useMemo

렌더링할 때 비용이 큰 계산이 있고 그 입력이 자주 바뀌지 않는다면 계산 결과를 재사용할 수 있습니다.

```jsx
const visibleItems = useMemo(() => {
  return items.filter((item) => item.title.includes(keyword));
}, [items, keyword]);
```

의존성인 `items`나 `keyword`가 바뀌면 다시 계산합니다. 단순한 문자열 연결이나 작은 배열 연산까지 감싸면 의존성 관리 비용과 코드 복잡도가 더 커질 수 있습니다.

## 2. 함수 참조를 보존하는 useCallback

함수는 렌더링마다 새 객체로 만들어집니다. 메모된 자식에 콜백을 전달하거나 다른 Hook의 의존성으로 사용할 때 안정된 함수 참조가 유용할 수 있습니다.

```jsx
const handleSelect = useCallback((id) => {
  setSelectedId(id);
}, []);
```

`useCallback(fn, deps)`는 함수의 실행 결과가 아니라 함수 자체를 보존합니다. 함수가 참조하는 Props나 State는 의존성 배열에 포함해야 최신 값을 사용합니다.

## 3. 렌더링과 무관한 값을 보존하는 useRef

ref 객체는 렌더링 사이에 같은 객체로 유지됩니다. DOM 요소에 접근하거나 화면 표시와 무관한 식별자·이전값을 저장할 때 사용할 수 있습니다.

```jsx
function FocusButton() {
  const inputRef = useRef(null);

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>입력으로 이동</button>
    </>
  );
}
```

화면에 보여야 하는 값은 State를 사용해야 합니다. ref의 `.current`를 바꿔도 React가 다시 렌더링하지 않기 때문입니다.

## 코드로 보기 — 검색 결과와 입력 포커스

```jsx
import { useMemo, useRef, useState } from "react";

function SearchPanel({ products }) {
  const [keyword, setKeyword] = useState("");
  const inputRef = useRef(null);

  const results = useMemo(() => {
    const normalized = keyword.trim().toLowerCase();
    return products.filter((product) =>
      product.name.toLowerCase().includes(normalized)
    );
  }, [products, keyword]);

  return (
    <section>
      <input ref={inputRef} value={keyword} onChange={(e) => setKeyword(e.target.value)} />
      <button onClick={() => inputRef.current?.focus()}>검색어 수정</button>
      <p>{results.length}개 결과</p>
    </section>
  );
}
```

### 코드 목적

검색 입력에 따라 파생 결과를 계산하고, 버튼으로 실제 입력 DOM에 포커스를 이동합니다.

### 코드 흐름

1. State가 검색어를 관리합니다.
2. `useMemo`가 상품 목록과 검색어가 바뀔 때만 필터 결과를 계산합니다.
3. `useRef`가 입력 DOM을 가리키고 클릭 시 `focus()`를 호출합니다.

### 실행 결과 해석

검색어가 바뀌면 결과 개수가 다시 계산됩니다. 포커스 이동은 ref로 DOM에 명령하지만 별도 State 변경은 필요하지 않습니다.

### 실무 연결

큰 목록의 필터링, 메모된 자식에 콜백 전달, 입력 포커스·스크롤·미디어 제어처럼 서로 다른 성격의 성능 및 DOM 작업에 사용합니다.

## 직접 해보기

1. 숫자 배열의 합을 `useMemo`로 계산해 보세요.
2. 선택한 `id`를 State에 저장하는 함수를 `useCallback`으로 감싸 보세요.
3. 버튼 클릭 시 입력값을 비우고 다시 포커스하도록 ref 코드를 확장해 보세요.

<details>
<summary>정답 보기</summary>

1. `useMemo(() => numbers.reduce((sum, n) => sum + n, 0), [numbers])`처럼 배열을 의존성으로 둡니다.
2. `useCallback((id) => setSelectedId(id), [])` 형태로 함수 참조를 보존할 수 있습니다.
3. `inputRef.current.value = ""` 뒤에 `inputRef.current.focus()`를 호출할 수 있습니다. 다만 제어 입력이라면 값을 비우는 일은 State setter로 처리해야 합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `useMemo` vs `useCallback` | 전자는 계산된 값을, 후자는 함수 참조를 보존합니다. |
| `useRef` vs `useState` | ref 변경은 렌더링을 일으키지 않고, State 변경은 화면 갱신을 예약합니다. |
| 정확성 vs 최적화 | 메모이제이션이 없어도 결과는 맞아야 하며, 최적화는 측정된 비용을 줄이는 보조 수단입니다. |

## 연결되는 개념

- Hook의 기본과 의존성은 [State Hook과 Effect Hook](04-state-and-effect-hooks.md)에서 확인하세요.
- 재사용 가능한 Hook으로 묶는 방법은 [커스텀 Hook 설계](06-custom-hooks.md)에서 이어집니다.
- 목록 파생값의 실제 예시는 [통합 Todo 애플리케이션 설계](07-todo-application-architecture.md)에서 확인하세요.

## 셀프 체크

- [ ] 세 Hook이 보존하는 대상을 구분할 수 있다.
- [ ] 의존성 배열에 필요한 값을 넣을 수 있다.
- [ ] 메모이제이션이 필요 없는 단순 계산을 구분할 수 있다.
- [ ] ref와 State의 렌더링 차이를 설명할 수 있다.
- [ ] ref로 DOM 요소의 메서드를 호출할 수 있다.

### 복습 질문 및 답변

**Q1. `useMemo`는 어떤 값을 반환하나요?**

<details>
<summary>답</summary>

전달한 계산 함수의 실행 결과를 반환합니다. 의존성이 같으면 이전 계산 결과를 재사용할 수 있습니다.

</details>

**Q2. `useCallback`에 전달한 함수가 State를 읽는다면 무엇을 확인해야 하나요?**

<details>
<summary>답</summary>

그 State가 의존성 배열에 포함됐는지 확인해야 합니다. 빠지면 함수가 오래된 값을 캡처할 수 있습니다.

</details>

**Q3. 화면에 보여야 하는 카운트를 ref에만 저장하면 왜 문제가 되나요?**

<details>
<summary>답</summary>

`.current` 변경은 재렌더링을 일으키지 않으므로 화면이 최신값으로 갱신되지 않습니다. 화면 데이터는 State가 적합합니다.

</details>

## 한 줄 정리

> `useMemo`는 값, `useCallback`은 함수, `useRef`는 렌더링과 무관한 참조를 보존하며 필요할 때만 선택적으로 사용합니다.
