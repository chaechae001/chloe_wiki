# 폼과 UI 상태 관리

> 입력 화면은 값을 읽는 것보다 제출 시점, 유효성, 버튼 상태, 사용자 피드백을 함께 맞추는 일이 더 중요합니다.

`submit` · `input` · `preventDefault` · `disabled` · `classList`

## 핵심요약

- 폼 제출 로직은 버튼의 `click`보다 폼의 `submit` 이벤트에서 처리합니다.
- JavaScript 검증을 수행하려면 필요한 경우 기본 제출을 먼저 막습니다.
- 텍스트·클래스·`disabled`·접근성 속성을 하나의 상태에서 함께 갱신합니다.
- 값 변경 감지에는 키보드만 보는 `keyup`보다 `input` 이벤트가 폭넓습니다.
- 스크롤이나 선택 효과도 이벤트와 명확한 상태 전환으로 구성할 수 있습니다.

## 1. 폼 제출과 입력값 처리

### 1) 정의

폼은 관련 입력을 묶고 제출 의미를 제공하는 HTML 구조입니다. `submit` 이벤트는 제출 버튼 클릭뿐 아니라 입력창에서 Enter를 누르는 제출도 함께 다룹니다.

### 2) 왜 필요한가

클릭 이벤트만 처리하면 키보드 제출을 놓칠 수 있습니다. 폼 단위로 검증하면 입력 경로와 무관하게 같은 규칙을 적용하고 접근성도 유지할 수 있습니다.

### 3) 핵심 흐름 재구성

1. 폼과 입력·메시지 요소를 선택합니다.
2. 폼의 `submit`을 감지합니다.
3. 검증이 필요하면 `preventDefault()`로 기본 제출을 보류합니다.
4. 값을 정리하고 유효성을 판단합니다.
5. 결과 메시지와 UI 상태를 갱신합니다.

### 4) 쉬운 예시

서류 제출 창구에서 접수 전에 필수 칸을 확인하는 것과 같습니다. 서류가 부족하면 제출을 잠시 멈추고 무엇을 고쳐야 하는지 알려 줍니다.

### 5) 코드 예시

```javascript
const form = document.querySelector("#nickname-form");
const input = document.querySelector("#nickname");
const result = document.querySelector("#result");

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const nickname = input.value.trim();

  if (nickname.length < 2) {
    result.textContent = "두 글자 이상 입력해 주세요.";
    input.setAttribute("aria-invalid", "true");
    return;
  }

  input.removeAttribute("aria-invalid");
  result.textContent = `${nickname}님, 반갑습니다.`;
  form.reset();
});
```

### 6) 헷갈리는 점

`preventDefault()`를 호출했다고 제출 처리가 자동으로 끝나는 것은 아닙니다. 기본 동작을 대신할 검증, 화면 갱신 또는 서버 요청을 직접 구현해야 합니다.

### 7) 한 줄 정리

> 폼은 `submit`을 중심으로 값 검증과 사용자 피드백을 하나의 흐름으로 묶습니다.

## 2. 입력에 따른 버튼과 선택 상태

### 1) 정의

UI 상태는 현재 입력이 유효한지, 어떤 항목이 선택됐는지, 동작이 진행 중인지처럼 화면이 표현해야 하는 조건입니다.

### 2) 왜 필요한가

색상만 바꾸고 실제 버튼은 눌리게 두면 화면과 동작이 어긋납니다. 상태 변화에서는 시각적 클래스, 동작 가능 여부, 보조기술용 속성을 함께 맞춰야 합니다.

### 3) 핵심 흐름 재구성

```javascript
function syncButton() {
  const ready = email.value.trim() !== "" && password.value.length >= 8;
  submitButton.disabled = !ready;
  submitButton.classList.toggle("is-ready", ready);
}
```

하나의 불리언 `ready`를 기준으로 여러 표현을 동기화하면 조건이 서로 엇갈리지 않습니다.

### 4) 쉬운 예시

신호등 색만 초록색으로 칠하고 실제 차단기를 열지 않으면 사용할 수 없습니다. 색, 상태, 안내 문구가 같은 조건을 바라봐야 합니다.

### 5) 코드 예시

```javascript
const fields = document.querySelectorAll("#login input");
const loginButton = document.querySelector("#login button");

function syncLoginState() {
  const complete = Array.from(fields).every((field) => field.value.trim() !== "");
  loginButton.disabled = !complete;
  loginButton.classList.toggle("is-active", complete);
}

fields.forEach((field) => field.addEventListener("input", syncLoginState));
syncLoginState();
```

### 6) 헷갈리는 점

`keyup`은 붙여넣기, 자동 완성, 음성 입력 같은 값 변화를 모두 포괄하지 못할 수 있습니다. 값이 바뀐 사실이 중요하면 `input` 이벤트가 보통 더 적합합니다.

### 7) 한 줄 정리

> 하나의 상태 값에서 클래스와 실제 동작 가능 여부를 함께 계산해야 UI가 일관됩니다.

## 코드로 보기 — 상태 메시지와 상단 이동 버튼

```javascript
const topButton = document.querySelector("#go-top");
const status = document.querySelector("#scroll-status");

topButton.addEventListener("click", () => {
  window.scrollTo({ top: 0, behavior: "smooth" });
  status.textContent = "페이지 상단으로 이동했습니다.";
});

window.addEventListener("scroll", () => {
  const visible = window.scrollY > 300;
  topButton.hidden = !visible;
});
```

### 코드 목적

일정 거리 이상 스크롤했을 때만 상단 이동 버튼을 보이고, 클릭 후 상태를 알립니다.

### 코드 흐름

1. 클릭 시 `window.scrollTo()`로 상단 이동을 요청합니다.
2. 상태 메시지를 갱신합니다.
3. 스크롤 위치에 따라 버튼의 `hidden` 상태를 계산합니다.

### 실행 결과 해석

페이지 상단에서는 버튼이 숨겨지고 300px보다 아래로 내려가면 표시됩니다. 클릭하면 부드럽게 상단으로 이동합니다.

### 실무 연결

긴 문서의 상단 이동, 회원가입 검증, 검색 조건 적용, 결제 버튼 활성화처럼 사용자 입력과 화면 상태가 함께 움직이는 기능에 적용됩니다.

## 직접 해보기

1. 폼의 기본 제출을 막는 코드를 작성해 보세요.
2. 두 입력칸이 모두 채워졌을 때만 버튼을 활성화하는 조건을 작성해 보세요.
3. `keyup` 대신 `input` 이벤트를 선택할 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. 제출 핸들러 안에서 `event.preventDefault()`를 호출합니다.
2. `const ready = first.value.trim() !== "" && second.value.trim() !== ""; button.disabled = !ready;`처럼 작성합니다.
3. 붙여넣기나 자동 완성처럼 키를 놓지 않아도 값이 바뀌는 입력 경로를 감지하기 위해서입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `click` vs `submit` | 특정 버튼 행동인지 폼 전체의 제출 시도인지 다릅니다. |
| `keyup` vs `input` | 키보드 동작인지 실제 값 변화인지 감지 기준이 다릅니다. |
| CSS 비활성 모양 vs `disabled` | 보이는 스타일과 실제 조작 가능 상태는 별개입니다. |
| `value` vs `textContent` | 폼 입력의 현재 값과 일반 요소의 텍스트 내용입니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [이벤트 대상과 위임](02-event-targets-and-delegation.md)
- 다음에 이어지는 개념: [동적 목록과 타이머 인터랙션](04-dynamic-lists-and-timers.md)
- 함께 보면 좋은 키워드: `검증`, `접근성`, `상태 동기화`

## 셀프 체크

- [ ] 폼의 `submit` 이벤트를 처리할 수 있다.
- [ ] 기본 동작 취소 뒤 필요한 로직을 구현할 수 있다.
- [ ] 입력값을 읽고 정리할 수 있다.
- [ ] 하나의 조건으로 버튼 상태와 클래스를 동기화할 수 있다.
- [ ] 값 변경 감지에 적절한 이벤트를 선택할 수 있다.

### 복습 질문 및 답변

**Q1. 버튼 클릭보다 폼 제출을 듣는 장점은 무엇인가요?**

<details><summary>답</summary>

버튼 클릭과 Enter 키 등 여러 제출 경로를 하나의 로직으로 처리할 수 있습니다.

</details>

**Q2. `form.reset()`은 무엇을 하나요?**

<details><summary>답</summary>

폼 컨트롤의 값을 초기 상태로 되돌립니다. 초기화 후 버튼 상태도 필요하면 다시 계산해야 합니다.

</details>

**Q3. 클래스만 바꿔 비활성화하면 부족한 이유는 무엇인가요?**

<details><summary>답</summary>

모양만 비활성처럼 보일 뿐 실제 클릭이나 키보드 조작은 계속 가능할 수 있기 때문입니다.

</details>

## 한 줄 정리

> 폼과 UI 상태는 이벤트, 값, 실제 조작 가능 여부, 피드백을 하나의 조건으로 동기화해야 합니다.

