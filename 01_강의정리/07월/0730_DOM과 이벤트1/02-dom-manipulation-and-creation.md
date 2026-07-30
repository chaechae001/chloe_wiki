# 요소 조작과 동적 생성

> DOM을 선택했다면 이제 내용과 상태를 바꾸고, 필요한 구조를 새로 만들어 화면에 연결할 차례입니다.

`textContent` · `classList` · `attribute` · `createElement` · `appendChild`

## 핵심요약

- 텍스트만 바꿀 때는 `textContent`, HTML 구조를 해석시킬 때는 `innerHTML`을 사용합니다.
- 상태에 따른 디자인은 인라인 스타일보다 CSS 클래스 추가·제거로 관리하기 좋습니다.
- `createElement()`는 메모리에만 요소를 만들며 삽입 메서드를 호출해야 화면에 나타납니다.
- DOM 메서드로 텍스트를 넣으면 사용자 입력을 HTML로 해석하지 않아 더 안전합니다.
- 삽입·삭제는 반드시 부모와 자식의 관계를 기준으로 수행합니다.

## 1. 내용·속성·스타일 변경

### 1) 정의

선택한 요소의 프로퍼티와 메서드를 사용하면 텍스트, HTML 속성, CSS 클래스, 인라인 스타일을 읽거나 변경할 수 있습니다.

### 2) 왜 필요한가

로그인 상태 표시, 버튼 비활성화, 경고 강조처럼 같은 HTML 구조에서도 애플리케이션 상태에 따라 화면 표현이 달라져야 합니다.

### 3) 핵심 흐름 재구성

| 작업 | 도구 | 예시 |
|---|---|---|
| 텍스트 변경 | `textContent` | `message.textContent = "저장됨"` |
| 속성 읽기·쓰기 | `getAttribute()`, `setAttribute()` | `input.setAttribute("aria-invalid", "true")` |
| 클래스 상태 변경 | `classList.add/remove/toggle()` | `card.classList.toggle("is-active")` |
| 간단한 인라인 스타일 | `style` | `bar.style.width = "70%"` |

### 4) 쉬운 예시

요소가 배우라면 텍스트는 대사, 속성은 이름표, 클래스는 의상, 인라인 스타일은 즉석에서 붙이는 소품과 같습니다. 반복되는 상태 표현은 의상 세트인 클래스로 관리하는 편이 일관적입니다.

### 5) 코드 예시

```javascript
const saveButton = document.querySelector("#save");
const status = document.querySelector("#status");

saveButton.disabled = true;
saveButton.classList.add("is-loading");
status.textContent = "저장 중…";
status.setAttribute("aria-live", "polite");
```

### 6) 헷갈리는 점

`innerHTML`은 `<strong>완료</strong>`를 실제 태그로 해석하지만 `textContent`는 그대로 글자로 표시합니다. 출처가 불확실한 입력을 `innerHTML`에 넣으면 악성 스크립트가 섞이는 XSS 위험이 있습니다.

### 7) 한 줄 정리

> 내용은 텍스트로, 상태는 클래스로, 필요한 속성은 명시적으로 변경하는 것이 기본입니다.

## 2. 요소 생성과 트리 연결

### 1) 정의

`createElement()`로 새 요소를 만든 뒤 텍스트와 속성을 설정하고, `appendChild()`나 `insertBefore()`로 기존 DOM 트리에 연결합니다.

### 2) 왜 필요한가

댓글, 알림, 장바구니 항목처럼 데이터에 따라 개수가 바뀌는 UI는 HTML에 전부 미리 적을 수 없습니다. 실행 중에 데이터를 요소로 바꾸는 과정이 필요합니다.

### 3) 핵심 흐름 재구성

1. 새 요소를 메모리에 만듭니다.
2. 텍스트·속성·클래스를 설정합니다.
3. 필요하면 자식 구조를 조립합니다.
4. 부모 요소에 삽입합니다.
5. 삭제할 때는 부모에서 해당 자식을 제거합니다.

### 4) 쉬운 예시

레고 부품을 책상 위에서 조립하는 것이 `createElement()` 단계라면, 완성한 조각을 전시판에 끼우는 것이 `appendChild()` 단계입니다.

### 5) 코드 예시

```javascript
const list = document.querySelector("#notice-list");
const item = document.createElement("li");
const label = document.createElement("strong");

label.textContent = "새 알림";
item.appendChild(label);
item.appendChild(document.createTextNode(" 주문이 준비되었습니다."));
item.classList.add("notice");
list.appendChild(item);
```

### 6) 헷갈리는 점

생성만 하고 삽입하지 않으면 화면에는 나타나지 않습니다. 또 `appendChild()`에 이미 DOM에 있는 노드를 넘기면 복사본이 생기는 것이 아니라 기존 노드가 새 위치로 이동합니다.

### 7) 한 줄 정리

> 동적 UI는 요소 생성, 내용 설정, 부모 연결의 세 단계로 만들어집니다.

## 코드로 보기 — 안전한 할 일 항목 추가

```javascript
function addTask(text) {
  const list = document.querySelector("#tasks");
  if (!list || text.trim() === "") return;

  const item = document.createElement("li");
  const title = document.createElement("span");
  const removeButton = document.createElement("button");

  title.textContent = text.trim();
  removeButton.type = "button";
  removeButton.textContent = "삭제";
  removeButton.setAttribute("aria-label", `${text.trim()} 삭제`);

  item.append(title, removeButton);
  list.appendChild(item);
}

addTask("DOM 복습하기");
```

### 코드 목적

입력 문자열을 HTML로 해석하지 않고 새 목록 항목과 삭제 버튼을 조립합니다.

### 코드 흐름

1. 부모 목록과 입력값을 검증합니다.
2. 필요한 요소를 각각 생성합니다.
3. `textContent`와 속성으로 내용을 채웁니다.
4. 자식을 항목에, 항목을 목록에 연결합니다.

### 실행 결과 해석

`#tasks`의 마지막에 “DOM 복습하기”와 삭제 버튼을 가진 항목이 추가됩니다. 입력에 `<b>`가 있어도 태그가 아니라 텍스트로 표시됩니다.

### 실무 연결

API에서 받은 댓글 목록, 검색 결과 카드, 대시보드 알림을 렌더링할 때 같은 방식으로 데이터와 DOM 생성을 분리합니다.

## 직접 해보기

1. `p` 요소를 만들고 “반갑습니다”라는 텍스트를 넣어 보세요.
2. 새 요소를 `#messages`의 첫 자식 앞에 삽입하는 순서를 적어 보세요.
3. 사용자 입력을 표시할 때 `innerHTML`보다 `textContent`가 안전한 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. `const p = document.createElement("p"); p.textContent = "반갑습니다";`입니다.
2. 부모와 기준 자식을 찾고 `parent.insertBefore(newNode, parent.firstElementChild)`를 호출합니다.
3. `textContent`는 문자열을 HTML로 해석하지 않아 삽입된 태그나 스크립트가 실행되지 않기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `textContent` vs `innerHTML` | 앞은 텍스트, 뒤는 HTML 구조로 해석합니다. |
| `style` vs `classList` | 단발성 값은 style로 가능하지만 반복되는 UI 상태는 클래스가 관리하기 쉽습니다. |
| `createElement()` vs `appendChild()` | 앞은 생성, 뒤는 DOM 트리에 연결하는 작업입니다. |
| `appendChild()` vs `insertBefore()` | 마지막 자식에 붙이는지 기준 자식 앞에 넣는지 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [DOM 트리와 요소 선택](01-dom-tree-and-selection.md)
- 다음에 이어지는 개념: [이벤트와 핸들러 등록](03-events-and-handlers.md)
- 함께 보면 좋은 키워드: `상태`, `렌더링`, `XSS`

## 셀프 체크

- [ ] 텍스트와 HTML 삽입의 차이를 설명할 수 있다.
- [ ] 클래스 추가·제거·토글을 사용할 수 있다.
- [ ] 새 요소를 만들어 부모에 연결할 수 있다.
- [ ] 기준 위치 앞에 요소를 삽입할 수 있다.
- [ ] 안전한 텍스트 출력 방법을 선택할 수 있다.

### 복습 질문 및 답변

**Q1. 만든 요소가 화면에 보이지 않는 가장 흔한 이유는 무엇인가요?**

<details><summary>답</summary>

`createElement()`만 호출하고 기존 DOM의 부모에 삽입하지 않았기 때문입니다.

</details>

**Q2. 체크 카드의 선택 상태를 표현하기 좋은 방법은 무엇인가요?**

<details><summary>답</summary>

CSS에 선택 상태 클래스를 정의하고 `element.classList.toggle("is-selected")`로 전환합니다.

</details>

**Q3. 삭제하려는 요소와 부모의 관계가 중요한 이유는 무엇인가요?**

<details><summary>답</summary>

`removeChild()`는 부모가 직접 가진 자식만 제거할 수 있기 때문입니다.

</details>

## 한 줄 정리

> DOM 조작은 선택한 요소의 상태를 바꾸거나, 새 객체를 만들어 정확한 부모 위치에 연결하는 작업입니다.

