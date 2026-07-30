# DOM 트리와 요소 선택

> 화면을 바꾸려면 먼저 HTML이 JavaScript에서 어떤 모습으로 존재하며, 바꿀 대상을 어떻게 찾는지 알아야 합니다.

`DOM` · `document` · `node` · `querySelector` · `NodeList`

## 핵심요약

- 브라우저는 HTML을 해석해 부모·자식 관계를 가진 DOM 트리를 만듭니다.
- `document`는 현재 문서 전체를 나타내는 탐색의 시작점입니다.
- `getElementById()`와 `querySelector()`는 요소 하나를 반환합니다.
- `querySelectorAll()` 등은 요소 모음을 반환하므로 반복이나 인덱스 접근이 필요합니다.
- 선택 결과가 없을 수 있으므로 조작 전에 `null` 여부를 확인하면 오류를 줄일 수 있습니다.

## 1. HTML이 객체 트리가 되는 과정

### 1) 정의

DOM(Document Object Model)은 HTML 문서를 JavaScript가 다룰 수 있는 객체들의 계층으로 표현한 모델입니다. 태그는 요소 노드가 되고, 태그 안의 글자는 텍스트 노드가 됩니다.

### 2) 왜 필요한가

HTML 문자열만으로는 실행 중인 화면의 제목을 바꾸거나 목록을 추가하기 어렵습니다. 브라우저가 각 부분을 객체로 제공하기 때문에 JavaScript는 특정 요소를 선택하고 프로퍼티와 메서드를 사용할 수 있습니다.

### 3) 핵심 흐름 재구성

```html
<main>
  <h1>오늘의 할 일</h1>
  <ul id="task-list">
    <li class="task">복습하기</li>
  </ul>
</main>
```

이 구조에서 `main`은 `h1`과 `ul`의 부모이고, `li`는 `ul`의 자식입니다. JavaScript는 `document`에서 출발해 `#task-list`나 `.task`를 찾습니다.

### 4) 쉬운 예시

DOM 트리는 건물 안내도와 비슷합니다. “건물 전체”가 `document`, 각 방이 node, 층과 복도가 부모·자식 관계입니다. 방을 고친다면 먼저 정확한 위치를 찾아야 합니다.

### 5) 코드 예시

```javascript
const list = document.getElementById("task-list");
const firstTask = document.querySelector("#task-list .task");

console.log(list);
console.log(firstTask.textContent);
```

### 6) 헷갈리는 점

HTML 원본 파일과 현재 DOM은 항상 같지 않습니다. JavaScript로 요소를 추가하거나 지우면 메모리의 DOM과 화면은 바뀌지만, 서버에 저장된 원본 HTML 파일이 수정되는 것은 아닙니다.

### 7) 한 줄 정리

> DOM은 브라우저가 HTML을 JavaScript용 객체 트리로 바꾼 결과입니다.

## 2. 하나 선택하기와 모두 선택하기

### 1) 정의

선택 메서드는 조건에 맞는 요소 하나 또는 요소 모음을 반환합니다.

| 목적 | 메서드 | 반환 특징 |
|---|---|---|
| 고유한 ID로 하나 선택 | `getElementById("title")` | 요소 또는 `null` |
| CSS 선택자로 첫 요소 선택 | `querySelector(".card")` | 요소 또는 `null` |
| 태그·클래스로 모두 선택 | `getElementsByTagName("li")` | HTMLCollection |
| CSS 선택자로 모두 선택 | `querySelectorAll(".card button")` | NodeList |

### 2) 왜 필요한가

단일 버튼의 텍스트를 바꾸는 작업과 모든 메뉴 항목에 이벤트를 등록하는 작업은 반환 형태가 다릅니다. 요소 모음에 곧바로 `style`을 적용하면 실패하므로, 몇 개가 선택되는지 먼저 판단해야 합니다.

### 3) 핵심 흐름 재구성

1. 원하는 대상을 CSS 선택자로 표현합니다.
2. 한 개인지 여러 개인지 결정합니다.
3. 반환값을 변수에 저장합니다.
4. `null` 또는 `length`를 확인합니다.
5. 여러 개라면 반복하며 각 요소를 사용합니다.

### 4) 쉬운 예시

학생 한 명을 학번으로 찾는 것은 단일 선택이고, 같은 동아리 학생 전체를 찾는 것은 다중 선택입니다. 학생 명단 자체에게 이름을 묻지 않고 명단 안의 각 학생에게 물어야 합니다.

### 5) 코드 예시

```javascript
const title = document.querySelector("main > h1");
const tasks = document.querySelectorAll("#task-list .task");

if (title) {
  title.textContent = `할 일 ${tasks.length}개`;
}

tasks.forEach((task, index) => {
  task.dataset.order = String(index + 1);
});
```

### 6) 헷갈리는 점

`getElementsByClassName("task")`에는 점을 붙이지 않지만 `querySelectorAll(".task")`에는 CSS 선택자 문법에 따라 점을 붙입니다.

### 7) 한 줄 정리

> 선택 메서드는 반환값이 요소 하나인지 모음인지까지 함께 기억해야 합니다.

## 코드로 보기 — 선택한 항목 요약하기

```javascript
const items = document.querySelectorAll("[data-price]");
let total = 0;

items.forEach((item) => {
  total += Number(item.dataset.price);
});

const output = document.querySelector("#total");
if (output) output.textContent = `${total.toLocaleString()}원`;
```

### 코드 목적

`data-price` 속성이 있는 모든 요소를 찾아 가격 합계를 화면에 표시합니다.

### 코드 흐름

1. 속성 선택자로 항목 모음을 찾습니다.
2. 각 문자열 값을 숫자로 바꿔 누적합니다.
3. 출력 요소가 존재할 때만 텍스트를 변경합니다.

### 실행 결과 해석

가격이 1200, 2500인 두 항목이 있다면 `#total`에는 `3,700원`이 표시됩니다. 목록은 다중 선택, 출력 칸은 단일 선택이라는 차이가 드러납니다.

### 실무 연결

상품 목록 집계, 필터 결과 개수, 선택된 체크 항목 요약처럼 여러 DOM 요소에서 값을 모으는 UI에 같은 흐름을 사용합니다.

## 직접 해보기

1. `id="notice"`인 요소 하나를 선택하는 코드를 작성해 보세요.
2. `.menu-item` 요소 전체의 개수를 출력해 보세요.
3. 선택자가 잘못되어 `null`이 반환될 때 오류를 막는 방법을 설명해 보세요.

<details>
<summary>정답 보기</summary>

1. `document.getElementById("notice")` 또는 `document.querySelector("#notice")`를 사용합니다.
2. `document.querySelectorAll(".menu-item").length`를 출력합니다.
3. `if (element) { ... }` 또는 옵셔널 체이닝으로 존재 여부를 확인한 뒤 조작합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| node vs element | node는 텍스트까지 포함하는 넓은 개념이고 element는 HTML 태그에 해당하는 노드입니다. |
| `querySelector()` vs `querySelectorAll()` | 첫 번째 하나를 반환하는지, 일치하는 전체 목록을 반환하는지 다릅니다. |
| `#id` vs `.class` | CSS 선택자에서 `#`은 ID, `.`은 클래스입니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: HTML 태그와 CSS 선택자
- 다음에 이어지는 개념: [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md)
- 함께 보면 좋은 키워드: `파싱`, `트리`, `컬렉션`

## 셀프 체크

- [ ] DOM과 HTML 파일의 관계를 설명할 수 있다.
- [ ] 단일 선택과 다중 선택을 구분할 수 있다.
- [ ] CSS 선택자로 원하는 범위를 표현할 수 있다.
- [ ] 선택 결과가 없을 때의 오류를 막을 수 있다.
- [ ] 요소 모음을 반복해 각 요소를 사용할 수 있다.

### 복습 질문 및 답변

**Q1. DOM 탐색의 시작점은 무엇인가요?**

<details><summary>답</summary>

현재 웹 문서 전체를 나타내는 `document` 객체입니다.

</details>

**Q2. 요소 모음에 바로 `style.color`를 지정할 수 없는 이유는 무엇인가요?**

<details><summary>답</summary>

모음은 단일 요소가 아니므로 각 요소를 인덱스나 반복문으로 꺼내 조작해야 합니다.

</details>

**Q3. 목록 내부 버튼만 모두 찾으려면 어떤 선택자가 적합한가요?**

<details><summary>답</summary>

예를 들어 `document.querySelectorAll("#task-list button")`처럼 부모 범위를 포함한 CSS 선택자를 사용합니다.

</details>

## 한 줄 정리

> DOM 작업의 첫 단계는 트리에서 반환 형태에 맞는 선택 메서드로 정확한 대상을 찾는 것입니다.

