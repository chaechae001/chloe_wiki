# HTTP 요청과 응답 메시지

API 통신 오류는 대부분 메서드, 주소, 헤더, 본문, 상태 코드 중 하나에서 단서를 찾을 수 있습니다. 메시지를 구성 요소로 나누면 네트워크 탭을 읽는 힘이 생깁니다.

## 핵심 키워드

`메서드` · `경로` · `헤더` · `본문` · `상태 코드` · `Content-Type`

## 핵심 요약

- 요청은 시작 줄, 헤더, 선택적인 본문으로 구성됩니다.
- 메서드는 자원에 수행할 동작을 나타냅니다.
- 응답 상태 코드는 처리 결과의 범주를 알려 줍니다.
- `Content-Type`은 본문의 데이터 형식을 설명합니다.

## 1. 요청 메시지

요청 시작 줄에는 메서드, 자원 경로, HTTP 버전이 들어갑니다. 헤더는 부가 정보를, 본문은 생성하거나 수정할 실제 데이터를 담습니다.

```http
POST /api/tasks HTTP/1.1
Host: service.example
Accept: application/json
Content-Type: application/json

{"title":"메시지 구조 복습","done":false}
```

| 요소 | 의미 | 확인할 질문 |
|---|---|---|
| 메서드 | 서버에 요청하는 작업 | 조회인가, 생성인가, 수정인가? |
| 경로 | 작업 대상 자원 | 올바른 자원과 식별자를 가리키는가? |
| 헤더 | 형식·인증 같은 부가 정보 | 서버가 기대하는 데이터 형식인가? |
| 본문 | 실제로 전달할 데이터 | 필드명과 값의 형식이 맞는가? |

대표 메서드는 `GET` 조회, `POST` 생성, `PUT` 전체 교체, `PATCH` 일부 수정, `DELETE` 삭제로 해석할 수 있습니다.

## 2. 응답 메시지

응답은 상태 줄, 헤더, 선택적인 본문으로 구성됩니다.

```http
HTTP/1.1 201 Created
Content-Type: application/json

{"id":7,"title":"메시지 구조 복습","done":false}
```

상태 코드는 성공과 실패의 종류를 빠르게 분류합니다.

| 범위 | 의미 | 예시 상황 |
|---|---|---|
| 1xx | 처리 진행 정보 | 다음 단계의 통신을 계속함 |
| 2xx | 성공 | 조회·생성·수정이 정상 처리됨 |
| 3xx | 다른 위치나 추가 조치 | 다른 주소로 이동해야 함 |
| 4xx | 요청 쪽 문제 | 잘못된 값, 인증 부족, 없는 자원 |
| 5xx | 서버 처리 문제 | 서버 내부 로직이나 의존 서비스 실패 |

상태 코드만으로 화면에 보여 줄 문구를 모두 결정하기보다, 응답 본문의 오류 코드나 메시지도 함께 확인하는 편이 좋습니다.

## 3. JSON 직렬화와 역직렬화

네트워크 본문은 문자열 형태로 이동합니다. 자바스크립트 객체를 JSON 문자열로 바꾸는 과정이 직렬화이고, JSON 문자열을 객체로 읽는 과정이 역직렬화입니다.

```javascript
const task = { title: "요청 본문 만들기", done: false };
const body = JSON.stringify(task);
const restored = JSON.parse(body);

console.log(body); // {"title":"요청 본문 만들기","done":false}
console.log(restored.title); // 요청 본문 만들기
```

JSON은 함수나 `undefined` 같은 모든 자바스크립트 값을 그대로 표현하지 못합니다. API 계약에 맞는 문자열, 숫자, 불리언, 배열, 객체, `null` 중심으로 데이터를 구성해야 합니다.

## 대표 코드: 응답을 안전하게 분류하기

### 목적

상태 코드 범위에 따라 사용자에게 보여 줄 결과를 구분합니다.

```javascript
function classifyResponse(response) {
  if (response.status >= 200 && response.status < 300) {
    return { ok: true, message: "요청이 처리되었습니다." };
  }

  if (response.status >= 400 && response.status < 500) {
    return { ok: false, message: "요청 내용을 확인해 주세요." };
  }

  if (response.status >= 500) {
    return { ok: false, message: "잠시 후 다시 시도해 주세요." };
  }

  return { ok: false, message: "추가 처리가 필요합니다." };
}

console.log(classifyResponse({ status: 404 }).message);
```

### 흐름과 결과

1. 응답 상태를 성공, 요청 오류, 서버 오류 범위로 나눕니다.
2. 내부 상태 코드를 그대로 노출하지 않고 상황에 맞는 안내를 만듭니다.
3. `404`는 4xx 범위이므로 요청 내용을 확인하라는 결과가 나옵니다.

### 실무 연결

개발자 도구의 Network 탭에서 Request Method, Request Headers, Request Payload, Status Code, Response를 같은 순서로 보면 문제의 위치를 좁힐 수 있습니다.

## 직접 해보기

`204`와 `503`을 `classifyResponse`에 넣었을 때 각각 어떤 결과가 나오는지 설명하세요.

<details>
<summary>답</summary>

`204`는 2xx라 성공 결과를 반환합니다. `503`은 5xx라 잠시 후 다시 시도하라는 결과를 반환합니다. `204`는 성공했지만 응답 본문이 없을 수 있다는 점도 함께 기억해야 합니다.

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| `Accept` vs `Content-Type` | Accept는 받고 싶은 응답 형식, Content-Type은 보내는 본문의 형식입니다. |
| `PUT` vs `PATCH` | PUT은 자원 전체 교체, PATCH는 일부 필드 변경에 주로 사용합니다. |
| 4xx vs 5xx | 4xx는 요청을 수정할 가능성이 크고, 5xx는 서버 처리 문제일 가능성이 큽니다. |

## 연결되는 개념

- 통신 참여자의 역할은 [클라이언트·서버와 HTTP 통신](01-client-server-and-http.md)에서 확인할 수 있습니다.
- 메서드와 경로를 일관되게 조합하는 법은 [REST API 설계](03-rest-api-design.md)에서 이어집니다.
- 실제 요청 코드는 [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md)에서 다룹니다.

## 셀프 체크

- [ ] 요청 메시지의 구성 요소를 구분할 수 있다.
- [ ] 상태 코드 범위로 성공과 오류를 분류할 수 있다.
- [ ] JSON 직렬화가 필요한 이유를 설명할 수 있다.

## 복습 질문 및 답변

### Q1. 요청 본문의 형식을 서버에 알려 주는 대표 헤더는 무엇인가?

<details>
<summary>답</summary>

`Content-Type`입니다. JSON을 보낼 때는 보통 `application/json`을 사용합니다.

</details>

### Q2. 인증 정보가 없어서 요청이 거절됐다면 어느 범주의 상태 코드일 가능성이 큰가?

<details>
<summary>답</summary>

클라이언트 요청 문제를 나타내는 4xx 범주일 가능성이 큽니다.

</details>

### Q3. 성공 상태인데 `response.json()`이 실패할 수 있는 경우는?

<details>
<summary>답</summary>

`204 No Content`처럼 본문이 없거나, 본문이 JSON이 아닌데 JSON으로 해석하려 할 때 실패할 수 있습니다.

</details>

## 한 줄 정리

> HTTP 메시지는 요청의 의도와 데이터, 응답의 처리 결과를 구조화해 전달하는 웹 통신의 봉투입니다.
