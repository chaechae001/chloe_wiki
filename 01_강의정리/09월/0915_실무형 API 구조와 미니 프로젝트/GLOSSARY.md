# 용어집

| 용어 | 설명 |
|---|---|
| Layered architecture | 책임과 의존성을 층으로 나눈 구조 |
| Router | HTTP path, request, response를 처리하는 계층 |
| Service | 프롬프트·캐시·후처리 같은 업무 규칙 계층 |
| Client | 외부 SDK와 API를 호출하는 adapter 계층 |
| Schema | 입력·출력 field와 validation 계약 |
| Dependency direction | 상위에서 하위로 흐르는 import·호출 방향 |
| Fail fast | 잘못된 설정을 가능한 이른 시점에 실패시키는 원칙 |
| System instruction | 모델 역할과 응답 규칙을 정하는 서버 정책 |
| Prompt template | 변수 자리를 가진 재사용 가능한 prompt 형식 |
| Request ID | 한 요청을 여러 레이어에서 추적하는 식별자 |
| TTFT | 첫 token이 나타날 때까지의 시간 |
| Structured logging | 고정 field 형식으로 남기는 log |
| Response cache | 완성된 응답을 앱이 저장하고 재사용하는 cache |
| Prompt caching | 공통 prompt prefix 처리 작업을 provider가 재사용하는 기능 |
| Cache key | 동일 요청 여부를 판단하는 결정론적 식별자 |
| TTL | Cache 항목이 유효한 시간 |
| Fake Client | 실제 네트워크 대신 예측 가능한 값을 반환하는 test double |

## 관련 문서

- [전체 개요](OVERVIEW.md)
- [실무형 레이어드 API 구조](01-layered-api-architecture.md)
- [응답 캐시와 미니 프로젝트](06-response-cache-and-mini-project.md)
