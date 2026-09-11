# OVERVIEW

이 자료는 백엔드 API의 기본 구조를 요청 전부터 응답 이후까지 하나의 흐름으로 이해하도록 구성했습니다. 단순히 endpoint를 호출하는 데 그치지 않고, 계약 설계·검증·오류 처리·테스트까지 연결합니다.

## 학습 목표

- API, backend, service의 역할과 경계를 설명한다.
- HTTP 요청과 응답을 구성 요소별로 읽는다.
- JSON 직렬화와 schema 검증의 관계를 이해한다.
- REST 관점에서 일관된 resource와 endpoint를 설계한다.
- Handler와 service를 분리하고 안전한 오류 응답을 만든다.
- Client 호출과 여러 층의 테스트로 API 계약을 검증한다.

## 학습 순서

1. [API, Backend, Service의 역할](01-api-backend-and-service.md) — API를 기능 목록이 아닌 계약으로 이해합니다.
2. [HTTP 요청과 응답 메시지](02-http-request-response.md) — method, path, header, body, status를 구분합니다.
3. [JSON과 데이터 직렬화](03-json-serialization.md) — 전송 데이터의 표현과 schema를 연결합니다.
4. [REST 리소스와 Endpoint 설계](04-rest-resource-design.md) — 예측 가능한 URL과 method 조합을 만듭니다.
5. [요청 처리 흐름과 입력 검증](05-request-lifecycle-validation.md) — 요청이 routing부터 service까지 이동하는 과정을 익힙니다.
6. [API 클라이언트 호출과 테스트](06-api-client-testing.md) — timeout, 실패 처리, 계약 테스트를 적용합니다.

용어가 낯설다면 [GLOSSARY](GLOSSARY.md)를 함께 확인하세요.

## 전체 흐름

```text
Client
  └─ HTTP Request: method + URL + headers + body
       └─ Router / Middleware
            └─ Authentication / Validation
                 └─ Handler
                      └─ Service / Repository / External API
                 └─ Serialization / Error mapping
       └─ HTTP Response: status + headers + body
  └─ Status 확인 / Parsing / Retry 판단 / Logging
```

## 핵심 설계 원칙

| 원칙 | 실천 방법 |
|---|---|
| 계약을 명시한다 | Schema, 상태 코드, 오류 code를 문서와 테스트로 고정한다. |
| HTTP 의미를 지킨다 | Resource와 method를 일관되게 사용하고 적절한 상태 코드를 선택한다. |
| 경계에서 검증한다 | 외부 입력은 신뢰하지 않고 형식·타입·권한·업무 규칙을 확인한다. |
| 책임을 분리한다 | Handler는 HTTP 경계, service는 업무 규칙을 담당한다. |
| 실패를 설계한다 | Timeout, 재시도, 오류 응답, request ID를 정상 흐름과 함께 정의한다. |
| 자동으로 검증한다 | Unit, integration, contract, E2E test를 목적에 맞게 조합한다. |

## 최종 점검 질문

1. 같은 URL에서 GET과 DELETE가 다른 endpoint인 이유는 무엇인가요?
2. `Content-Type`과 `Accept`는 각각 누구의 어떤 의도를 나타내나요?
3. JSON 문법 오류와 업무 규칙 오류는 어느 계층에서 구분해야 하나요?
4. 201, 202, 204는 어떤 상황에서 선택하나요?
5. Handler와 service를 분리하면 테스트가 어떻게 달라지나요?
6. Client가 재시도하기 전에 확인해야 할 HTTP method의 성질은 무엇인가요?

<details>
<summary>답</summary>

Method까지 포함해야 작업 의미가 정해집니다. `Content-Type`은 현재 body 형식, `Accept`는 원하는 응답 형식입니다. JSON 문법·schema는 요청 경계에서, 외부 상태가 필요한 업무 규칙은 service에서 검사합니다. 생성 완료는 201, 처리 접수는 202, 반환 body 없는 성공은 204가 적합합니다. 계층을 분리하면 업무 규칙을 HTTP 환경 없이 단위 테스트할 수 있습니다. 재시도 전에는 idempotent 여부를 확인해야 합니다.

</details>

## 완료 기준

- [ ] 여섯 개 문서를 순서대로 읽었다.
- [ ] 각 문서의 실습을 직접 풀었다.
- [ ] 요청과 응답 예시를 한 개씩 작성했다.
- [ ] 성공·검증 실패·권한 실패·서버 실패 테스트를 만들었다.
- [ ] GLOSSARY의 핵심 용어를 자신의 말로 설명했다.
