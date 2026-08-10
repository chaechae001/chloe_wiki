# Axios 요청 패턴

Axios는 HTTP 요청 설정과 응답 데이터 처리를 일관된 객체 형태로 제공하는 라이브러리입니다. 편리함을 얻는 대신 응답 구조와 오류 구조를 정확히 이해해야 합니다.

## 핵심 키워드

`Axios` · `응답 객체` · `data` · `구조 분해` · `then` · `async/await`

## 핵심 요약

- Axios는 브라우저 기본 API가 아닌 별도 라이브러리입니다.
- 요청 Promise의 성공값은 `data`, `status`, `headers` 등을 가진 응답 객체입니다.
- `then`과 `async/await`은 같은 Promise를 다른 문법으로 소비합니다.
- 공통 설정과 오류 변환을 한곳에 모으면 호출부가 단순해집니다.

## 1. 응답 객체와 구조 분해

Axios의 성공 응답에서 실제 서버 본문은 보통 `response.data`에 들어 있습니다.

```javascript
async function readTasks(api) {
  const response = await api.get("/tasks");
  console.log(response.status);
  return response.data;
}
```

필요한 `data`만 사용할 때는 구조 분해 할당으로 의도를 더 분명히 표현할 수 있습니다.

```javascript
async function readTasks(api) {
  const { data } = await api.get("/tasks");
  return data;
}
```

이 문법이 네트워크 동작을 바꾸는 것은 아닙니다. 응답 객체에서 `data` 속성을 꺼내는 자바스크립트 문법일 뿐입니다.

## 2. then과 async/await

```javascript
function readWithThen(api) {
  return api.get("/tasks").then((response) => response.data);
}

async function readWithAwait(api) {
  const response = await api.get("/tasks");
  return response.data;
}
```

두 함수 모두 작업 결과를 Promise로 반환합니다. 짧은 변환은 `then`도 읽기 쉽고, 여러 순차 단계와 `try/catch`가 필요한 흐름은 `async/await`이 자연스러운 경우가 많습니다.

## 3. 요청 설정

생성 요청에서는 본문 객체를 두 번째 인수로 전달하는 형태가 일반적입니다.

```javascript
async function createTask(api, title) {
  const payload = { title, done: false };
  const { data } = await api.post("/tasks", payload);
  return data;
}
```

라이브러리가 JSON 직렬화와 응답 JSON 해석을 도와주더라도 서버와 합의한 필드명, 인증 헤더, 오류 처리는 개발자가 설계해야 합니다.

## 대표 코드: API 모듈 만들기

### 목적

화면 코드에서 주소와 응답 구조를 숨기고 자원 중심의 함수만 노출합니다.

```javascript
function createTaskApi(client) {
  return {
    async list() {
      const { data } = await client.get("/tasks");
      return data;
    },

    async create(title) {
      const { data } = await client.post("/tasks", {
        title,
        done: false,
      });
      return data;
    },

    async complete(id) {
      const { data } = await client.patch(`/tasks/${id}`, { done: true });
      return data;
    },
  };
}
```

### 흐름과 결과

1. 설정된 Axios 클라이언트를 외부에서 전달받습니다.
2. `list`, `create`, `complete`가 주소와 메서드를 담당합니다.
3. 각 함수는 응답 전체가 아니라 화면에 필요한 `data`를 반환합니다.
4. 화면은 `taskApi.complete(id)`처럼 도메인 동작에 집중할 수 있습니다.

### 실무 연결

기본 URL, 인증 헤더, 제한 시간을 가진 인스턴스와 응답 인터셉터를 사용하면 반복 설정을 줄일 수 있습니다. 다만 인터셉터가 숨은 동작을 늘릴 수 있으므로 팀 규칙과 오류 형태를 문서화해야 합니다.

## 4. 오류를 사용자 메시지로 바꾸기

네트워크 오류 객체를 그대로 화면에 노출하지 말고 진단 정보와 사용자 안내를 분리합니다.

```javascript
function toUserMessage(error) {
  const status = error.response?.status;

  if (status === 404) return "요청한 항목을 찾지 못했습니다.";
  if (status && status >= 500) return "서버가 응답하지 못했습니다.";
  if (error.request) return "네트워크 연결을 확인해 주세요.";
  return "요청을 준비하는 중 문제가 발생했습니다.";
}
```

## 직접 해보기

`createTaskApi`에 할 일을 삭제하는 `remove(id)` 메서드를 추가해 보세요.

<details>
<summary>답</summary>

```javascript
async function removeTask(client, id) {
  await client.delete(`/tasks/${id}`);
  return id;
}
```

삭제 응답에 본문이 없을 수 있으므로 호출한 식별자를 반환하게 설계할 수 있습니다. 실제 반환 규칙은 API 계약에 맞춰야 합니다.

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| Fetch vs Axios | Fetch는 브라우저 기본 API이고 Axios는 별도 라이브러리이며 응답·오류 처리 기본값이 다릅니다. |
| `response` vs `response.data` | response는 상태·헤더를 포함한 전체 응답이고 data는 서버가 보낸 본문입니다. |
| `then` vs `await` | 둘 다 Promise 결과를 다루며 표현 방식과 오류 처리 구조가 다를 뿐입니다. |

## 연결되는 개념

- REST 자원과 메서드 규칙은 [자원 중심의 REST API 설계](03-rest-api-design.md)에서 확인할 수 있습니다.
- 브라우저 기본 요청 도구는 [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md)에서 다룹니다.
- API 모듈의 결과를 화면에 쓰는 법은 [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md)에서 이어집니다.

## 셀프 체크

- [ ] Axios 응답에서 본문을 찾을 수 있다.
- [ ] 구조 분해 할당의 역할을 설명할 수 있다.
- [ ] 요청 코드를 자원별 모듈로 분리할 수 있다.

## 복습 질문 및 답변

### Q1. Axios는 브라우저에 기본 내장된 API인가?

<details>
<summary>답</summary>

아닙니다. 프로젝트에 설치하거나 별도로 불러와 사용하는 라이브러리입니다.

</details>

### Q2. `const { data } = await client.get(url)`은 무엇을 줄인 표현인가?

<details>
<summary>답</summary>

응답 객체를 받은 뒤 그 객체의 `data` 속성을 꺼내는 두 단계를 구조 분해로 합친 표현입니다.

</details>

### Q3. API 호출 함수를 화면 이벤트 처리기와 분리하면 어떤 장점이 있는가?

<details>
<summary>답</summary>

주소·메서드·응답 구조의 변경을 한곳에서 관리하고 여러 화면에서 같은 요청 로직을 재사용하고 테스트하기 쉬워집니다.

</details>

## 한 줄 정리

> Axios는 요청과 응답을 편리하게 다루게 해 주지만, 응답 객체와 오류 구조를 이해하고 API 모듈로 경계를 세워야 효과적입니다.
