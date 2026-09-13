# 용어집

> OpenAI API와 FastAPI 기반 LLM endpoint를 이해하는 데 필요한 핵심 용어입니다.

| 용어 | 설명 |
|---|---|
| API key | 외부 API 호출 권한을 증명하는 비밀 자격 증명 |
| Backend proxy | 클라이언트 대신 외부 서비스를 호출하는 서버 경계 |
| Base URL | API 요청 path의 기준이 되는 주소 |
| Responses API | 입력과 도구·출력을 통합해 다루는 OpenAI API |
| Chat Completions | `messages` 대화 배열을 중심으로 한 API 형식 |
| `instructions` | 모델의 역할과 응답 규칙을 지정하는 입력 |
| `input` | 이번 요청에서 모델이 처리할 내용 |
| `output_text` | SDK가 response output에서 텍스트를 모아 제공하는 helper |
| Token | 모델이 텍스트를 처리하는 기본 단위 |
| Stateless | 요청 사이의 상태를 자동으로 기억하지 않는 성질 |
| Response ID | 생성 결과를 식별하고 지원되는 상태 연결에 사용하는 값 |
| Pydantic | Python type을 이용해 데이터 schema와 validation을 제공하는 도구 |
| Response model | API가 반환할 field와 type을 제한하는 schema |
| 400 | 형식은 처리 가능하지만 의미상 잘못된 요청에 주로 쓰는 상태 |
| 422 | Request schema validation 실패에 FastAPI가 사용하는 상태 |
| 429 | 요청 한도 초과를 나타내는 상태 |
| 502 | Upstream 서비스로부터 유효한 처리를 얻지 못했음을 나타내는 상태 |
| 504 | Upstream 응답을 제한 시간 안에 받지 못했음을 나타내는 상태 |
| Timeout | 외부 호출의 최대 대기 시간 |
| Backoff | 재시도 사이의 대기 시간을 늘리는 정책 |
| Request ID | 개별 요청을 로그와 지원 과정에서 추적하는 식별자 |
| Liveness | 애플리케이션 process가 살아 있는지 확인하는 상태 |
| Readiness | 애플리케이션이 실제 요청 처리 준비가 되었는지 확인하는 상태 |
| Contract test | Status, schema, 오류 형태 등 API 약속을 검증하는 test |
| Secret rotation | 기존 자격 증명을 폐기하고 새 값으로 교체하는 절차 |

## 연결해서 기억하기

```text
Secret 관리
  → FastAPI request validation
  → LLM service 호출
  → Responses output 정리
  → 오류 mapping과 logging
  → Contract test
```

## 셀프 체크

- [ ] API key와 사용자 인증을 구분한다.
- [ ] 400, 422, 429, 502, 504의 역할을 비교한다.
- [ ] Timeout과 backoff를 설명한다.
- [ ] Liveness와 readiness를 구분한다.
- [ ] Contract test의 목적을 설명한다.

## 관련 문서

- [전체 개요](OVERVIEW.md)
- [API 키와 서버 보안](01-api-key-and-server-security.md)
- [오류 처리·재시도·관측성](05-errors-retries-and-observability.md)
