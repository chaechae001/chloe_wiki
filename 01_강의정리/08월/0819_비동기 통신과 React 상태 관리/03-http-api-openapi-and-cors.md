# HTTP API와 OpenAPI, CORS

> 프런트엔드의 비동기 코드는 결국 ‘어떤 요청을 보내고 어떤 응답을 약속받는가’라는 API 계약 위에서 동작합니다.

`HTTP` · `endpoint` · `OpenAPI` · `Origin` · `CORS`

## 핵심요약

- API 요청은 method, endpoint, header, query, path, body로 구성됩니다.
- 응답은 상태 코드와 payload를 함께 해석해야 합니다.
- API 테스트 도구는 UI와 서버 문제를 분리해 확인하게 돕습니다.
- OpenAPI는 요청과 응답의 형태를 기계가 읽을 수 있게 정의합니다.
- CORS 허용 여부는 브라우저와 서버가 Origin을 기준으로 판단합니다.

## 1. 요청과 응답 읽기

```javascript
async function createPost(input) {
  const response = await fetch('/api/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(input),
  });

  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

### 코드 목적

JavaScript 객체를 JSON 요청 본문으로 보내고 HTTP 상태를 확인한 뒤 응답 데이터를 반환합니다.

### 코드 흐름과 결과 해석

method와 content type을 지정하고 직렬화한 body를 전송합니다. `fetch`는 404나 500 응답만으로 reject되지 않으므로 `response.ok`를 직접 확인합니다. 성공하면 JSON 파싱 결과가 다음 상태 업데이트에 사용됩니다.

### 실무 연결

화면 개발 전 API 테스트 도구로 method, endpoint, 인증 header, query, body를 먼저 검증하면 UI 오류와 서버 오류를 분리하기 쉽습니다. 관련 요청은 Collection으로 묶고 환경별 base URL을 변수로 관리할 수 있습니다.

## 2. OpenAPI가 만드는 계약

OpenAPI 문서는 endpoint별 method, 인증 방식, query·path 변수, 요청 body, 응답 schema와 상태 코드를 정의합니다. 프런트엔드와 백엔드는 같은 문서를 기준으로 구현과 테스트를 진행하고, 문서 UI나 클라이언트 코드 생성 도구가 이를 활용할 수 있습니다.

## 3. Origin과 CORS

Origin은 프로토콜, 호스트, 포트의 조합입니다. 브라우저에서 다른 Origin으로 요청하면 서버가 응답 헤더로 해당 Origin을 허용했는지 확인합니다. 필요한 경우 실제 요청 전에 `OPTIONS` preflight가 method와 header 허용 여부를 점검합니다.

CORS 오류는 프런트 코드가 응답 헤더를 임의로 추가해 해결하는 문제가 아닙니다. 서버 또는 프록시가 신뢰할 Origin, method, header, credentials 정책을 정확히 설정해야 합니다.

## 직접 해보기

1. POST 요청에 JSON body를 보낼 때 필요한 header를 적으세요.
2. OpenAPI schema가 협업에 주는 이점을 설명하세요.
3. 개발 서버와 API 서버의 포트가 다를 때 CORS 관점에서 무엇이 달라지는지 말하세요.

<details>
<summary>답</summary>

1. 일반적으로 `Content-Type: application/json`을 지정합니다.
2. 요청과 응답 구조를 하나의 계약으로 공유해 구현 차이와 수동 문서 오류를 줄입니다.
3. 포트가 다르면 Origin도 달라져 서버의 CORS 허용 정책 대상이 됩니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| path parameter vs query | 자원 식별 경로 vs 정렬·필터 같은 선택 조건 |
| HTTP 오류 vs 네트워크 오류 | 서버가 오류 상태로 응답함 vs 요청 자체가 완료되지 못함 |
| OpenAPI vs API 테스트 도구 | 계약 형식 vs 실제 요청 실행·저장 도구 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [async/await와 Promise 조합](02-async-await-and-promise-combinators.md)
- 다음에 이어지는 개념: [React 상태 관리와 Context](04-react-state-and-context.md)

## 셀프 체크

- [ ] HTTP 요청의 구성 요소를 구분한다.
- [ ] `fetch`에서 HTTP 상태를 직접 검사한다.
- [ ] OpenAPI 문서의 역할을 설명한다.
- [ ] Origin의 세 구성 요소를 말할 수 있다.
- [ ] CORS 정책을 서버 관점에서 이해한다.

### 복습 질문 및 답변

**Q1. `fetch`가 404 응답에서 항상 catch로 이동하는가?**

<details>
<summary>답</summary>

아닙니다. 응답 자체를 받았으면 Promise가 fulfilled될 수 있으므로 `response.ok`나 status를 검사해야 합니다.

</details>

**Q2. preflight 요청의 목적은?**

<details>
<summary>답</summary>

브라우저가 실제 교차 출처 요청 전에 서버가 method와 header 등을 허용하는지 확인하는 것입니다.

</details>

**Q3. 모든 Origin을 허용하면 언제 위험한가?**

<details>
<summary>답</summary>

인증 정보나 민감한 작업을 다루는 API에서 신뢰하지 않는 사이트까지 브라우저 요청을 허용할 수 있어 위험합니다.

</details>

## 한 줄 정리

> 안정적인 비동기 통신은 HTTP 요청 구성, 명시적인 API 계약, 정확한 교차 출처 정책을 함께 이해하는 데서 시작합니다.
