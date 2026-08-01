# 동적 목록과 타이머 인터랙션

> 동적으로 변하는 화면은 요소를 만드는 코드뿐 아니라 이벤트 연결과 종료 조건까지 함께 설계해야 완성됩니다.

`dynamic list` · `filter` · `delegation` · `setInterval` · `cleanup`

## 핵심요약

- 사용자 입력은 새 요소로 변환해 DOM 목록에 연결할 수 있습니다.
- 동적 삭제는 부모 목록의 이벤트 위임으로 처리하기 좋습니다.
- 필터 입력은 비교 기준을 정규화한 뒤 각 항목의 표시 상태를 갱신합니다.
- 반복 움직임은 `setInterval()`로 만들 수 있지만 반드시 종료·초기화 로직이 필요합니다.
- 여러 타이머가 같은 결과를 결정할 때 중복 종료를 막는 상태가 필요합니다.

## 1. 생성·삭제·필터가 있는 동적 목록

### 1) 정의

동적 목록은 사용자의 입력이나 외부 데이터에 따라 실행 중 항목이 추가·삭제되고, 검색 조건에 따라 보이는 항목이 바뀌는 UI입니다.

### 2) 왜 필요한가

할 일, 장바구니, 댓글, 알림처럼 항목 개수가 고정되지 않은 화면은 데이터에서 DOM을 만들고 다시 사용자 행동을 데이터 변화에 연결해야 합니다.

### 3) 핵심 흐름 재구성

1. 폼 제출을 감지하고 빈 값을 검사합니다.
2. `li`와 버튼을 생성해 안전하게 텍스트를 넣습니다.
3. 부모 목록에 새 항목을 추가합니다.
4. 부모의 클릭 리스너에서 삭제 버튼을 판별합니다.
5. 검색어와 항목 텍스트를 같은 규칙으로 바꾼 뒤 표시 여부를 정합니다.

### 4) 쉬운 예시

화이트보드에 할 일을 붙이고, 완료된 쪽지를 떼고, 특정 단어가 있는 쪽지만 남겨 보는 과정과 같습니다. 보드는 부모 목록, 쪽지는 동적 항목입니다.

### 5) 코드 예시

```javascript
const form = document.querySelector("#add-form");
const input = document.querySelector("#new-item");
const list = document.querySelector("#items");

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const value = input.value.trim();
  if (!value) return;

  const item = document.createElement("li");
  const label = document.createElement("span");
  const removeButton = document.createElement("button");

  label.textContent = value;
  removeButton.type = "button";
  removeButton.dataset.remove = "true";
  removeButton.textContent = "삭제";

  item.append(label, removeButton);
  list.appendChild(item);
  form.reset();
});
```

### 6) 헷갈리는 점

새 항목마다 삭제 리스너를 등록할 수도 있지만, 목록 부모에 이미 위임 리스너가 있다면 새 항목도 자동으로 처리됩니다. 생성 로직과 이벤트 등록을 불필요하게 결합하지 않아도 됩니다.

### 7) 한 줄 정리

> 동적 목록은 생성은 데이터에서, 행동은 부모 위임에서 처리하면 구조가 단순해집니다.

## 2. 타이머 기반 움직임과 정리

### 1) 정의

`setInterval()`은 일정 간격마다 함수를 호출해 위치나 숫자를 조금씩 변경합니다. 반환된 타이머 ID를 `clearInterval()`에 전달하면 반복을 중단할 수 있습니다.

### 2) 왜 필요한가

진행 표시, 카운트다운, 간단한 게임처럼 시간에 따라 상태가 반복해서 변하는 기능은 시작·반복·종료의 수명주기를 가져야 합니다.

### 3) 핵심 흐름 재구성

1. 시작 시 중복 실행을 막습니다.
2. 반복마다 현재 위치를 읽고 다음 값을 계산합니다.
3. DOM 스타일이나 텍스트를 갱신합니다.
4. 종료 조건을 검사합니다.
5. 모든 관련 타이머를 정리하고 UI를 초기화합니다.

### 4) 쉬운 예시

주방 타이머를 시작했다면 시간이 끝났을 때 벨을 끄고 다시 시작할 수 있도록 초기화해야 합니다. 벨을 끄지 않으면 새 타이머와 이전 타이머가 동시에 울립니다.

### 5) 코드 예시

```javascript
const runner = document.querySelector("#runner");
const startButton = document.querySelector("#start-race");
let timerId = null;

function stopRace() {
  clearInterval(timerId);
  timerId = null;
  startButton.disabled = false;
}

startButton.addEventListener("click", () => {
  if (timerId !== null) return;
  startButton.disabled = true;

  let position = 0;
  timerId = setInterval(() => {
    position += 8;
    runner.style.transform = `translateX(${position}px)`;
    if (position >= 320) stopRace();
  }, 50);
});
```

### 6) 헷갈리는 점

`clearInterval()`은 타이머 ID 변수 자체를 `null`로 바꾸지 않습니다. 실행 상태를 코드에서 판단하려면 중단 뒤 변수도 명시적으로 초기화해야 합니다.

### 7) 한 줄 정리

> 타이머 인터랙션은 움직임보다 중복 시작 방지와 종료 정리가 더 중요합니다.

## 코드로 보기 — 삭제 위임과 실시간 필터

```javascript
const itemList = document.querySelector("#items");
const filterInput = document.querySelector("#filter");

itemList.addEventListener("click", (event) => {
  const button = event.target.closest("button[data-remove]");
  if (!button || !itemList.contains(button)) return;
  button.closest("li")?.remove();
});

filterInput.addEventListener("input", (event) => {
  const query = event.target.value.trim().toLowerCase();

  itemList.querySelectorAll("li").forEach((item) => {
    const label = item.querySelector("span")?.textContent.toLowerCase() ?? "";
    item.hidden = !label.includes(query);
  });
});
```

### 코드 목적

현재와 미래의 목록 항목을 삭제하고, 검색어가 포함된 항목만 실시간으로 표시합니다.

### 코드 흐름

1. 목록 클릭에서 삭제 버튼을 찾아 해당 `li`를 제거합니다.
2. 검색어를 공백 제거와 소문자 변환으로 정규화합니다.
3. 각 항목의 표시 텍스트도 소문자로 바꿉니다.
4. `includes()` 결과에 따라 `hidden`을 설정합니다.

### 실행 결과 해석

삭제 버튼을 누른 항목만 제거되고, 검색창에 입력하면 대소문자와 관계없이 검색어를 포함한 항목만 남습니다.

### 실무 연결

장바구니, 태그 관리, 관리자 목록, 간단한 검색 UI에 적용할 수 있습니다. 데이터가 많아지면 DOM 전체를 매번 순회하기보다 서버 검색이나 지연 처리를 고려합니다.

## 직접 해보기

1. 빈 문자열이 목록에 추가되지 않도록 조건을 작성해 보세요.
2. 반복 타이머를 종료하는 데 필요한 두 단계를 적어 보세요.
3. 필터 비교 전에 양쪽 문자열을 소문자로 바꾸는 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. `const value = input.value.trim(); if (!value) return;`처럼 검사합니다.
2. `clearInterval(timerId)`로 중단하고 상태 판단용 ID 변수를 `null`로 초기화합니다.
3. 대소문자 차이를 무시하고 사용자가 기대하는 일관된 포함 검색을 하기 위해서입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 생성 시 직접 등록 vs 부모 위임 | 새 항목마다 리스너를 추가하는지 기존 부모 하나가 처리하는지 다릅니다. |
| `setInterval()` vs `clearInterval()` | 반복 시작과 해당 반복의 중단입니다. |
| 화면 숨김 vs DOM 삭제 | `hidden`은 요소를 유지하고 숨기며 `remove()`는 트리에서 제거합니다. |
| 현재 위치 vs 다음 위치 | 종료 조건은 갱신 전후 어느 값을 검사하는지 일관되어야 합니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [폼과 UI 상태 관리](03-forms-and-ui-state.md)
- 다음에 이어지는 개념: 애니메이션 프레임과 비동기 처리
- 함께 보면 좋은 키워드: `렌더링`, `수명주기`, `검색 최적화`

## 셀프 체크

- [ ] 사용자 입력에서 안전하게 목록 항목을 만들 수 있다.
- [ ] 부모 위임으로 동적 항목을 삭제할 수 있다.
- [ ] 문자열을 정규화해 목록을 필터링할 수 있다.
- [ ] 반복 타이머를 시작하고 종료할 수 있다.
- [ ] 중복 실행과 중복 종료를 막는 상태를 설계할 수 있다.

### 복습 질문 및 답변

**Q1. 동적으로 추가된 삭제 버튼도 위임 리스너가 처리할 수 있는 이유는 무엇인가요?**

<details><summary>답</summary>

클릭 이벤트가 부모 목록으로 버블링되며 부모는 실제 클릭 대상을 검사하기 때문입니다.

</details>

**Q2. 시작 버튼을 타이머 실행 중 비활성화하는 이유는 무엇인가요?**

<details><summary>답</summary>

여러 반복 타이머가 중첩되어 위치 계산과 종료 처리가 충돌하는 것을 막기 위해서입니다.

</details>

**Q3. 복잡한 화면 애니메이션에서 `setInterval()` 대신 고려할 API는 무엇인가요?**

<details><summary>답</summary>

브라우저 화면 갱신 주기에 맞춰 호출되는 `requestAnimationFrame()`을 고려할 수 있습니다.

</details>

## 한 줄 정리

> 동적 목록과 시간 기반 UI는 생성·이벤트·상태·정리의 전체 수명주기를 함께 설계해야 안정적으로 동작합니다.

