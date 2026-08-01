# 이벤트 대상과 위임

> 클릭된 요소와 핸들러가 붙은 요소를 구분하면, 부모 리스너 하나로 수많은 자식을 다룰 수 있습니다.

`target` · `currentTarget` · `closest` · `contains` · `delegation`

## 핵심요약

- `event.target`은 실제 이벤트가 시작된 요소입니다.
- `event.currentTarget`은 현재 실행 중인 리스너가 등록된 요소입니다.
- 버블링을 이용하면 부모에서 자식들의 이벤트를 처리할 수 있습니다.
- `closest()`로 클릭 지점에서 의미 있는 자식 요소를 찾습니다.
- 위임 처리에서는 대상이 부모 범위 안에 있는지 검증해야 합니다.

## 1. target과 currentTarget

### 1) 정의

`target`은 이벤트의 최초 발생 지점이며 전파 중에도 바뀌지 않습니다. `currentTarget`은 현재 호출된 핸들러가 연결된 요소이므로 경로를 따라 핸들러가 실행될 때마다 달라질 수 있습니다.

### 2) 왜 필요한가

버튼 안에 아이콘이나 글자가 있을 때 사용자가 실제로 누른 노드는 아이콘일 수 있습니다. 하지만 실행하려는 동작은 전체 버튼의 데이터와 관련될 수 있어 두 대상을 구분해야 합니다.

### 3) 핵심 흐름 재구성

```html
<ul id="actions">
  <li><button data-action="save"><span>저장</span></button></li>
</ul>
```

`span`을 클릭하면 부모 `ul`의 핸들러에서 `event.target`은 `span`, `event.currentTarget`은 `ul`입니다.

### 4) 쉬운 예시

민원 접수 창구에서 민원을 실제 제기한 사람이 `target`이라면, 현재 서류를 처리 중인 부서가 `currentTarget`입니다. 민원인은 그대로지만 처리 부서는 단계마다 달라질 수 있습니다.

### 5) 코드 예시

```javascript
const actions = document.querySelector("#actions");

actions.addEventListener("click", (event) => {
  console.log("시작점", event.target);
  console.log("리스너 위치", event.currentTarget);
});
```

### 6) 헷갈리는 점

일반 함수 리스너 안의 `this`는 보통 `currentTarget`과 같지만 화살표 함수는 자신의 `this`를 만들지 않습니다. 이벤트 요소를 명확히 사용할 때는 `event.currentTarget`이 읽기 쉽습니다.

### 7) 한 줄 정리

> `target`은 사건의 출발점이고 `currentTarget`은 지금 사건을 처리하는 위치입니다.

## 2. 이벤트 위임

### 1) 정의

이벤트 위임은 자식에서 버블링되는 이벤트를 부모 하나에서 받아, 실제 대상을 판별하고 필요한 동작을 실행하는 패턴입니다.

### 2) 왜 필요한가

항목마다 리스너를 붙이면 요소가 많을수록 등록 코드와 관리 비용이 늘어납니다. 실행 중 새로 추가된 자식에는 기존 반복 등록이 적용되지 않지만 부모 리스너는 새 자식의 버블링 이벤트도 받을 수 있습니다.

### 3) 핵심 흐름 재구성

1. 공통 부모에 리스너 하나를 등록합니다.
2. `event.target.closest()`로 의미 있는 자식을 찾습니다.
3. 대상이 부모 내부인지 확인합니다.
4. 데이터 속성이나 클래스에 따라 동작을 분기합니다.

### 4) 쉬운 예시

반 전체 학생에게 안내원을 한 명씩 배치하는 대신 담임 한 명이 학생들의 요청을 받아 이름표를 확인하고 처리하는 방식과 비슷합니다.

### 5) 코드 예시

```javascript
const toolbar = document.querySelector("#toolbar");

toolbar.addEventListener("click", (event) => {
  const button = event.target.closest("button[data-action]");
  if (!button || !toolbar.contains(button)) return;

  const action = button.dataset.action;
  console.log(`${action} 동작 실행`);
});
```

### 6) 헷갈리는 점

`event.target.matches("button")`만 확인하면 버튼 안의 아이콘을 클릭했을 때 실패합니다. `closest("button")`를 사용하면 클릭 지점부터 조상 방향으로 실제 제어 요소를 찾을 수 있습니다.

### 7) 한 줄 정리

> 이벤트 위임은 버블링과 대상 판별을 결합해 동적 자식들을 부모 하나에서 처리합니다.

## 코드로 보기 — 목록의 선택과 삭제 위임하기

```javascript
const list = document.querySelector("#task-list");

list.addEventListener("click", (event) => {
  const deleteButton = event.target.closest("button[data-delete]");
  if (deleteButton && list.contains(deleteButton)) {
    deleteButton.closest("li")?.remove();
    return;
  }

  const item = event.target.closest("li[data-task-id]");
  if (!item || !list.contains(item)) return;

  item.classList.toggle("is-done");
  item.setAttribute("aria-checked", String(item.classList.contains("is-done")));
});
```

### 코드 목적

목록 부모의 리스너 하나로 항목 완료 상태와 삭제 버튼을 모두 처리합니다.

### 코드 흐름

1. 삭제 버튼을 먼저 찾고 해당 항목을 제거합니다.
2. 삭제가 아니면 할 일 항목을 찾습니다.
3. 부모 목록에 포함된 항목인지 검증합니다.
4. 완료 클래스와 접근성 상태를 함께 갱신합니다.

### 실행 결과 해석

항목 글자를 누르면 완료 상태가 토글되고 삭제 버튼을 누르면 해당 항목만 제거됩니다. 이후 JavaScript로 추가된 항목도 같은 부모 리스너가 처리합니다.

### 실무 연결

메일 목록, 데이터 테이블, 카드 그리드, 알림 센터처럼 항목이 자주 추가·삭제되는 화면에 적합합니다.

## 직접 해보기

1. 부모 리스너에서 실제 클릭된 요소를 얻는 프로퍼티를 적어 보세요.
2. 버튼 내부 아이콘 클릭도 버튼 동작으로 처리하는 코드를 작성해 보세요.
3. 동적 항목에 이벤트 위임이 유리한 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. `event.target`입니다.
2. `const button = event.target.closest("button");`처럼 가장 가까운 버튼을 찾습니다.
3. 새 항목마다 리스너를 다시 등록하지 않아도 부모로 버블링된 이벤트를 처리할 수 있기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `target` vs `currentTarget` | 실제 발생 요소와 현재 리스너가 붙은 요소의 차이입니다. |
| 직접 등록 vs 이벤트 위임 | 자식마다 리스너를 두는지 부모 하나에서 분기하는지 다릅니다. |
| `matches()` vs `closest()` | 자기 자신만 검사하는지 조상 방향까지 탐색하는지 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [이벤트 전파의 세 단계](01-event-propagation-phases.md)
- 다음에 이어지는 개념: [폼과 UI 상태 관리](03-forms-and-ui-state.md)
- 함께 보면 좋은 키워드: `데이터 속성`, `동적 DOM`, `접근성`

## 셀프 체크

- [ ] `target`과 `currentTarget`을 예제로 구분할 수 있다.
- [ ] 이벤트 위임이 버블링을 이용함을 설명할 수 있다.
- [ ] `closest()`로 의미 있는 제어 요소를 찾을 수 있다.
- [ ] 부모 범위 검증이 필요한 이유를 안다.
- [ ] 동적 항목의 선택과 삭제를 위임할 수 있다.

### 복습 질문 및 답변

**Q1. 자식의 아이콘을 클릭하면 부모 리스너의 `currentTarget`은 무엇인가요?**

<details><summary>답</summary>

리스너를 등록한 부모 요소입니다. 실제 클릭된 아이콘은 `target`입니다.

</details>

**Q2. 이벤트 위임에서 삭제 동작을 먼저 분기한 이유는 무엇인가요?**

<details><summary>답</summary>

삭제 버튼 클릭이 항목 선택 토글까지 동시에 실행되지 않도록 의도를 분리하기 위해서입니다.

</details>

**Q3. `contains()` 검사는 무엇을 막아 주나요?**

<details><summary>답</summary>

찾아낸 요소가 실제로 리스너의 관리 범위인 부모 내부에 있는지 확인해 잘못된 대상을 처리하는 일을 막습니다.

</details>

## 한 줄 정리

> 이벤트의 시작점과 처리 위치를 구분하면 부모 리스너 하나로 변화하는 자식 UI를 안정적으로 제어할 수 있습니다.

