# Ajax와 Fetch API

Ajax는 특정 라이브러리 이름이 아니라 페이지를 새로고침하지 않고 서버와 데이터를 교환해 화면 일부를 갱신하는 방식입니다. Fetch API는 이 흐름을 Promise로 표현하는 현대적인 브라우저 도구입니다.

## 핵심 키워드

`Ajax` · `XMLHttpRequest` · `Fetch API` · `Promise` · `Response` · `response.ok`

## 핵심 요약

- Ajax는 비동기 통신으로 필요한 화면만 갱신하는 기술적 접근입니다.
- `XMLHttpRequest`는 상태와 이벤트를 직접 관리하는 전통적인 API입니다.
- `fetch`는 Promise를 반환하지만 HTTP 오류 상태를 자동으로 reject하지 않습니다.
- Fetch 응답 본문은 `json()` 같은 비동기 메서드로 읽습니다.

## 1. Ajax와 XMLHttpRequest

Ajax의 이름에는 XML이 들어가지만 현재는 JSON을 더 자주 사용합니다. 핵심은 전체 문서를 다시 받지 않고 필요한 데이터만 요청해 현재 화면을 유지하는 것입니다.

`XMLHttpRequest`는 요청 상태 변화를 이벤트로 감시합니다.

```javascript
function loadTasksWithXhr() {
  const xhr = new XMLHttpRequest();
  xhr.open("GET", "/api/tasks", true);

  xhr.addEventListener("load", () => {
    if (xhr.status >= 200 && xhr.status < 300) {
      const tasks = JSON.parse(xhr.responseText);
      console.log(tasks);
    }
  });

  xhr.addEventListener("error", () => {
    console.error("네트워크 연결을 확인해 주세요.");
  });

  xhr.send();
}
```

요청 객체 생성, 연결 설정, 이벤트 등록, 상태 검사, JSON 변환을 직접 작성하므로 세밀한 제어가 가능하지만 코드가 길어지기 쉽습니다.

## 2. Fetch의 두 단계 응답 처리

`fetch()`가 이행되면 먼저 상태와 헤더를 가진 `Response` 객체를 받습니다. 실제 JSON 본문은 `response.json()`을 다시 기다려야 합니다.

```javascript
async function loadTasks() {
  const response = await fetch("/api/tasks");

  if (!response.ok) {
    throw new Error(`요청 실패: ${response.status}`);
  }

  const tasks = await response.json();
  return tasks;
}
```

이 코드는 다음 두 비동기 단계를 분리합니다.

1. 서버로부터 응답 헤더가 도착할 때까지 기다립니다.
2. 응답 본문을 읽고 JSON 객체로 해석할 때까지 기다립니다.

## 3. POST 요청

객체를 보낼 때는 JSON 문자열로 직렬화하고 형식을 헤더에 표시합니다.

```javascript
async function createTask(title) {
  const response = await fetch("/api/tasks", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, done: false }),
  });

  if (!response.ok) {
    throw new Error(`생성 실패: ${response.status}`);
  }

  return response.json();
}
```

`fetch`는 404나 500 응답을 받아도 네트워크 왕복 자체가 완료되면 보통 fulfilled Promise가 됩니다. 따라서 `response.ok` 또는 `status` 검사를 명시해야 합니다.

## 대표 코드: 로딩·성공·실패 상태 관리

### 목적

통신 결과뿐 아니라 사용자가 보는 로딩과 오류 상태까지 함께 관리합니다.

```javascript
async function requestTasks(view) {
  view.showLoading();

  try {
    const response = await fetch("/api/tasks");

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const tasks = await response.json();
    view.showTasks(tasks);
  } catch (error) {
    view.showError("목록을 불러오지 못했습니다.");
    console.error(error);
  } finally {
    view.hideLoading();
  }
}
```

### 흐름과 결과

1. 요청 전에 로딩 표시를 켭니다.
2. 응답 상태와 JSON 변환을 차례로 확인합니다.
3. 성공하면 목록, 실패하면 사용자용 오류를 표시합니다.
4. 어떤 결과든 `finally`에서 로딩 표시를 끕니다.

### 실무 연결

버튼 중복 클릭 방지, 스켈레톤 화면, 재시도 버튼도 같은 로딩·성공·실패 상태 모델 위에 설계할 수 있습니다.

## 직접 해보기

`createTask("")`처럼 빈 제목이 들어오면 네트워크 요청 전에 오류를 던지도록 검증을 추가해 보세요.

<details>
<summary>답</summary>

```javascript
async function createTask(title) {
  if (!title.trim()) {
    throw new Error("제목을 입력해 주세요.");
  }

  // 검증을 통과한 뒤 fetch 요청을 수행합니다.
}
```

클라이언트 검증은 빠른 피드백을 주지만 서버에서도 같은 규칙을 반드시 검증해야 합니다.

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| Ajax vs Fetch | Ajax는 비동기 화면 갱신 방식이고 Fetch는 그 통신을 구현하는 브라우저 API입니다. |
| XHR vs Fetch | XHR은 이벤트·상태 중심이고 Fetch는 Promise 중심입니다. |
| 네트워크 실패 vs HTTP 오류 | 연결 실패는 reject될 수 있지만 404·500은 응답이므로 `response.ok` 검사가 필요합니다. |

## 연결되는 개념

- 응답 상태와 본문은 [HTTP 요청과 응답 메시지](02-http-messages.md)에서 설명합니다.
- Promise 흐름을 간결하게 감싼 도구는 [Axios 요청 패턴](05-axios-request-patterns.md)에서 이어집니다.
- 받은 데이터를 화면에 반영하는 법은 [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md)에서 다룹니다.

## 셀프 체크

- [ ] Ajax와 Fetch의 관계를 설명할 수 있다.
- [ ] Fetch 응답을 두 단계로 처리하는 이유를 안다.
- [ ] HTTP 오류 상태를 명시적으로 검사할 수 있다.

## 복습 질문 및 답변

### Q1. Ajax를 사용하면 항상 XML을 주고받아야 하는가?

<details>
<summary>답</summary>

아닙니다. 이름에 XML이 남아 있지만 현재는 JSON을 비롯한 다양한 형식을 사용할 수 있습니다.

</details>

### Q2. `await fetch()` 직후 곧바로 실제 데이터 배열을 얻는가?

<details>
<summary>답</summary>

아닙니다. 먼저 `Response` 객체를 받고 `await response.json()`처럼 본문을 별도로 읽어야 합니다.

</details>

### Q3. Fetch에서 404 응답을 `catch`로 보내려면 무엇이 필요한가?

<details>
<summary>답</summary>

`response.ok`나 상태 코드를 검사하고 직접 오류를 던져야 합니다.

</details>

## 한 줄 정리

> Ajax는 새로고침 없는 데이터 교환 방식이고 Fetch는 응답 상태와 본문을 Promise로 다루는 구현 도구입니다.
