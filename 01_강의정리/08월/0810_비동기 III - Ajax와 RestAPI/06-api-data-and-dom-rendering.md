# API 데이터와 DOM 렌더링

API 호출은 데이터를 받은 순간 끝나지 않습니다. 입력을 검증하고 필요한 항목을 고른 뒤 DOM 변경을 최소화해 사용자에게 안정적으로 보여 줘야 하나의 기능이 완성됩니다.

## 핵심 키워드

`이벤트` · `preventDefault` · `find` · `filter` · `DOM` · `렌더링`

## 핵심 요약

- 이벤트 처리기는 입력 수집, 통신, 데이터 가공, 렌더링 순서로 읽을 수 있어야 합니다.
- 폼 제출의 기본 새로고침은 필요할 때 `preventDefault()`로 막습니다.
- 하나를 찾을 때는 `find`, 여러 개를 고를 때는 `filter`가 의도를 잘 드러냅니다.
- 반복마다 DOM을 수정하지 말고 결과를 모아 한 번에 반영합니다.

## 1. 이벤트에서 통신 시작하기

버튼 클릭이나 선택값 변경은 API 호출의 시작점이 됩니다. 폼 안의 제출 버튼은 기본적으로 페이지 이동을 일으킬 수 있으므로 현재 화면에서 비동기 처리를 이어 갈 때 기본 동작을 막습니다.

```javascript
form.addEventListener("submit", async (event) => {
  event.preventDefault();
  // 입력 검증 → API 요청 → 화면 반영
});
```

`preventDefault()`는 모든 이벤트에 습관적으로 붙이는 코드가 아닙니다. 브라우저의 기본 동작을 대체할 때만 사용합니다.

## 2. 받은 데이터 찾기와 거르기

```javascript
const members = [
  { code: "A1", name: "하늘", active: true },
  { code: "B2", name: "나무", active: false },
  { code: "C3", name: "바다", active: true },
];

const selected = members.find((member) => member.code === "B2");
const activeMembers = members.filter((member) => member.active);

console.log(selected.name); // 나무
console.log(activeMembers.length); // 2
```

- `find`는 첫 번째 일치 객체 또는 `undefined`를 반환합니다.
- `filter`는 모든 일치 항목을 담은 새 배열을 반환합니다.
- 직접 반복문을 쓸 때는 찾은 뒤 `break`로 끝낼 수 있지만, 의도가 맞다면 배열 메서드가 더 분명합니다.

## 3. DOM 변경 횟수 줄이기

반복문 안에서 `innerHTML`을 계속 변경하면 브라우저가 화면 갱신 작업을 여러 번 수행할 수 있습니다. 먼저 메모리에서 결과를 만들고 마지막에 한 번 반영합니다.

```javascript
function renderMembers(listElement, members) {
  const items = members
    .map((member) => `<li>${member.name}</li>`)
    .join("");

  listElement.innerHTML = items;
}
```

외부에서 받은 문자열을 그대로 `innerHTML`에 넣으면 보안 문제가 생길 수 있습니다. 신뢰할 수 없는 데이터는 `textContent`와 `createElement`를 활용하는 편이 안전합니다.

```javascript
function renderMembersSafely(listElement, members) {
  const fragment = document.createDocumentFragment();

  for (const member of members) {
    const item = document.createElement("li");
    item.textContent = member.name;
    fragment.append(item);
  }

  listElement.replaceChildren(fragment);
}
```

## 대표 코드: 기준값으로 목록 필터링하기

### 목적

사용자 입력을 숫자로 검증하고 API 데이터에서 조건에 맞는 항목만 한 번에 렌더링합니다.

```javascript
async function showProductsAbovePrice({ form, input, list, api }) {
  form.addEventListener("submit", async (event) => {
    event.preventDefault();

    const minimum = Number(input.value);
    if (!Number.isFinite(minimum) || minimum < 0) {
      list.textContent = "0 이상의 가격을 입력해 주세요.";
      return;
    }

    try {
      const products = await api.listProducts();
      const matched = products.filter((product) => product.price >= minimum);

      const fragment = document.createDocumentFragment();
      for (const product of matched) {
        const item = document.createElement("li");
        item.textContent = `${product.name}: ${product.price.toLocaleString()}원`;
        fragment.append(item);
      }

      list.replaceChildren(fragment);
    } catch (error) {
      list.textContent = "상품 목록을 불러오지 못했습니다.";
      console.error(error);
    }
  });
}
```

### 흐름과 결과

1. 폼의 기본 제출을 막습니다.
2. 입력 문자열을 숫자로 바꾸고 유효성을 검사합니다.
3. API 결과에서 기준 이상 상품만 `filter`로 고릅니다.
4. 문서 조각에 항목을 모은 뒤 목록을 한 번에 교체합니다.
5. 실패하면 기술 오류와 사용자 안내를 분리합니다.

### 실무 연결

검색 결과, 대시보드 목록, 자동 완성, 선택값에 따른 상세 패널도 같은 이벤트 → 요청 → 가공 → 렌더링 흐름으로 만들 수 있습니다.

## 직접 해보기

검색 결과가 없을 때 `검색 결과가 없습니다.`라는 한 항목을 보여 주도록 코드를 보완해 보세요.

<details>
<summary>답</summary>

```javascript
if (matched.length === 0) {
  const emptyItem = document.createElement("li");
  emptyItem.textContent = "검색 결과가 없습니다.";
  list.replaceChildren(emptyItem);
  return;
}
```

빈 배열도 truthy이므로 배열 자체가 아니라 `length`를 확인해야 합니다.

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| `find` vs `filter` | find는 첫 번째 항목 하나 또는 undefined, filter는 조건을 만족한 모든 항목의 배열을 반환합니다. |
| `input.value` vs 숫자 | 입력값은 문자열일 수 있으므로 계산과 비교 전에 명시적으로 변환하는 편이 안전합니다. |
| `innerHTML` vs `textContent` | innerHTML은 마크업을 해석하고 textContent는 문자열을 텍스트로 처리합니다. |

## 연결되는 개념

- 비동기 요청의 성공과 실패는 [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md)에서 확인할 수 있습니다.
- 재사용 가능한 요청 계층은 [Axios 요청 패턴](05-axios-request-patterns.md)에서 다룹니다.
- 전체 학습 순서는 [학습 안내](OVERVIEW.md)에서 확인하세요.

## 셀프 체크

- [ ] 이벤트의 기본 동작을 막아야 하는 상황을 구분할 수 있다.
- [ ] find와 filter의 반환값 차이를 설명할 수 있다.
- [ ] DOM 변경을 반복문 밖으로 모을 수 있다.

## 복습 질문 및 답변

### Q1. 폼 제출 이벤트에서 페이지가 새로고침되지 않게 하는 메서드는?

<details>
<summary>답</summary>

이벤트 객체의 `preventDefault()`입니다.

</details>

### Q2. 고유 코드로 객체 하나를 찾을 때 적합한 배열 메서드는?

<details>
<summary>답</summary>

`find`입니다. 찾지 못하면 `undefined`가 되므로 사용 전에 확인해야 합니다.

</details>

### Q3. 외부 API의 이름을 화면에 안전하게 표시하려면 어떤 방식을 고려해야 하는가?

<details>
<summary>답</summary>

문자열을 `innerHTML`로 해석시키기보다 요소를 만들고 `textContent`로 값을 설정하는 방식을 고려합니다.

</details>

## 한 줄 정리

> API 기반 화면은 이벤트, 입력 검증, 데이터 선택, 최소 DOM 갱신, 오류 안내를 하나의 흐름으로 설계해야 합니다.
