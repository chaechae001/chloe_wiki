# event 객체와 상호작용 패턴

> 같은 클릭이라도 어디서, 어떤 입력으로 발생했는지 알아야 정확한 반응을 만들 수 있습니다.

`event.target` · `event.key` · `preventDefault` · `removeEventListener` · `interaction`

## 핵심요약

- 브라우저는 핸들러를 호출할 때 발생 정보를 담은 event 객체를 전달합니다.
- `event.target`은 실제 이벤트가 시작된 요소입니다.
- 키보드 이벤트의 `event.key`로 사용자가 누른 키를 구분할 수 있습니다.
- `preventDefault()`는 폼 제출이나 링크 이동 같은 기본 동작을 취소합니다.
- 리스너를 제거하려면 등록 때 사용한 것과 동일한 함수 참조가 필요합니다.

## 1. event 객체로 발생 정보 읽기

### 1) 정의

event 객체는 이벤트 종류, 발생 대상, 마우스 좌표, 키보드 입력 같은 정보와 이벤트를 제어하는 메서드를 담습니다. 핸들러의 첫 번째 매개변수로 받을 수 있습니다.

### 2) 왜 필요한가

목록의 어느 버튼을 눌렀는지, Enter 키인지 Escape 키인지, 폼 값을 검증한 뒤 제출할지 판단하려면 발생한 사건의 구체적인 정보가 필요합니다.

### 3) 핵심 흐름 재구성

```javascript
function handleClick(event) {
  console.log(event.type);   // "click"
  console.log(event.target); // 실제 클릭된 요소
}
```

핸들러 매개변수 이름은 `event`, `e` 등 자유롭게 정할 수 있지만 객체의 역할을 드러내는 이름이 읽기 쉽습니다.

### 4) 쉬운 예시

택배 알림이 이벤트라면 event 객체는 발신자, 도착 시간, 배송 위치가 적힌 송장입니다. 알림만으로는 세부 행동을 정하기 어렵지만 송장을 보면 어느 주문인지 판단할 수 있습니다.

### 5) 코드 예시

```javascript
const searchInput = document.querySelector("#search");

searchInput.addEventListener("keydown", (event) => {
  if (event.key === "Enter") {
    console.log(`검색: ${event.target.value}`);
  }
});
```

### 6) 헷갈리는 점

`event.target`은 실제로 클릭된 가장 안쪽 요소이고, `event.currentTarget`은 현재 핸들러가 등록되어 실행 중인 요소입니다. 버튼 안의 아이콘을 클릭하면 둘이 달라질 수 있습니다.

### 7) 한 줄 정리

> event 객체는 “무슨 일이 어디서 어떻게 일어났는가”를 핸들러에 전달합니다.

## 2. 기본 동작 제어와 리스너 제거

### 1) 정의

브라우저가 원래 수행하는 링크 이동, 폼 제출 등의 행동을 기본 동작이라고 합니다. `preventDefault()`는 이를 취소하며, `removeEventListener()`는 등록된 핸들러를 해제합니다.

### 2) 왜 필요한가

폼 값을 먼저 검증하거나 JavaScript로 비동기 제출하려면 즉시 새로고침되는 기본 제출을 막아야 합니다. 일회성 안내나 화면 해제 시에는 불필요한 리스너를 제거할 수 있어야 합니다.

### 3) 핵심 흐름 재구성

```javascript
function submitForm(event) {
  event.preventDefault();
  // 검증과 요청 로직
}

form.addEventListener("submit", submitForm);
form.removeEventListener("submit", submitForm);
```

등록과 제거에 같은 `submitForm` 참조를 사용합니다. 각각 새로 만든 익명 함수는 내용이 같아도 서로 다른 함수 객체입니다.

### 4) 쉬운 예시

기본 동작은 자동문처럼 조건 없이 열리는 행동입니다. `preventDefault()`는 잠시 자동 기능을 멈추고 신분 확인을 거친 뒤 우리가 원하는 흐름으로 진행하는 것과 같습니다.

### 5) 코드 예시

```javascript
const form = document.querySelector("#signup");
const email = document.querySelector("#email");
const message = document.querySelector("#form-message");

function validateForm(event) {
  event.preventDefault();

  if (!email.value.includes("@")) {
    message.textContent = "이메일 형식을 확인해 주세요.";
    email.setAttribute("aria-invalid", "true");
    return;
  }

  email.removeAttribute("aria-invalid");
  message.textContent = "입력 확인이 완료되었습니다.";
}

form.addEventListener("submit", validateForm);
```

### 6) 헷갈리는 점

`preventDefault()`는 이벤트 전파를 멈추는 메서드가 아닙니다. 기본 동작 취소와 부모로 전달되는 이벤트 흐름 제어는 서로 다른 문제입니다.

### 7) 한 줄 정리

> 기본 동작을 취소할 때는 이유와 후속 동작을 함께 설계해야 합니다.

## 코드로 보기 — 목록에서 클릭된 항목 토글하기

```javascript
const list = document.querySelector("#filter-list");

function handleFilterClick(event) {
  const button = event.target.closest("button[data-filter]");
  if (!button || !list.contains(button)) return;

  button.classList.toggle("is-active");
  button.setAttribute(
    "aria-pressed",
    String(button.classList.contains("is-active"))
  );
}

list.addEventListener("click", handleFilterClick);
```

### 코드 목적

여러 필터 버튼 각각에 리스너를 붙이지 않고 부모 하나에서 클릭을 처리합니다.

### 코드 흐름

1. 목록 부모에서 클릭 이벤트를 받습니다.
2. 실제 클릭 지점에서 가장 가까운 필터 버튼을 찾습니다.
3. 목록 바깥이거나 버튼이 아니면 종료합니다.
4. 활성 클래스를 토글하고 접근성 상태를 동기화합니다.

### 실행 결과 해석

버튼의 글자나 아이콘을 눌러도 해당 필터 버튼의 `is-active` 상태가 바뀝니다. 새 필터 버튼이 나중에 추가되어도 부모 리스너가 처리할 수 있습니다.

### 실무 연결

동적 목록, 테이블 행의 액션 버튼, 채팅 메시지 메뉴처럼 항목 수가 변하는 화면에서 이벤트 위임 패턴을 사용할 수 있습니다.

## 직접 해보기

1. 링크 클릭 시 이동을 막는 핸들러를 작성해 보세요.
2. `event.target`과 `event.currentTarget`의 차이를 설명해 보세요.
3. 익명 함수로 등록한 리스너를 같은 모양의 새 익명 함수로 제거할 수 없는 이유를 말해 보세요.

<details><summary>정답 보기</summary>

1. `link.addEventListener("click", (event) => event.preventDefault());`처럼 작성할 수 있습니다.
2. `target`은 실제 발생 지점, `currentTarget`은 현재 핸들러가 등록된 요소입니다.
3. 두 익명 함수는 코드 모양이 같아도 메모리에서 서로 다른 함수 객체이기 때문입니다. 제거할 핸들러는 이름 있는 변수에 보관합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `target` vs `currentTarget` | 실제 발생 요소와 리스너가 등록된 현재 요소의 차이입니다. |
| `preventDefault()` vs 이벤트 전파 제어 | 기본 브라우저 행동을 막는 것과 부모 방향 전달을 제어하는 것은 별개입니다. |
| `mouseover` vs `mouseout` | 포인터가 요소 안으로 들어올 때와 밖으로 나갈 때 발생합니다. |
| 등록 vs 제거 | 제거하려면 이벤트 이름뿐 아니라 동일한 함수 참조도 일치해야 합니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [이벤트와 핸들러 등록](03-events-and-handlers.md)
- 다음에 이어지는 개념: 이벤트 전파와 이벤트 위임
- 함께 보면 좋은 키워드: `접근성`, `폼 검증`, `상태 동기화`

## 셀프 체크

- [ ] event 객체에서 발생 대상과 키 정보를 읽을 수 있다.
- [ ] `target`과 `currentTarget`을 구분할 수 있다.
- [ ] 필요한 상황에서만 기본 동작을 취소할 수 있다.
- [ ] 같은 함수 참조로 리스너를 제거할 수 있다.
- [ ] 클래스와 ARIA 상태를 함께 갱신할 수 있다.

### 복습 질문 및 답변

**Q1. 폼 검증에서는 어떤 이벤트를 듣는 것이 좋은가요?**

<details><summary>답</summary>

버튼의 `click`만 보기보다 폼의 `submit` 이벤트를 처리하면 Enter 제출 등 여러 제출 경로를 함께 다룰 수 있습니다.

</details>

**Q2. 키보드에서 Enter를 구분하는 값은 무엇인가요?**

<details><summary>답</summary>

키보드 이벤트 객체의 `event.key === "Enter"`를 확인합니다.

</details>

**Q3. 이벤트 위임의 핵심 이점은 무엇인가요?**

<details><summary>답</summary>

부모 하나의 리스너로 여러 자식과 나중에 추가되는 자식의 이벤트까지 처리할 수 있다는 점입니다.

</details>

## 한 줄 정리

> event 객체를 읽고 기본 동작과 리스너 수명을 제어하면 단순 클릭을 안정적인 상호작용 흐름으로 바꿀 수 있습니다.

