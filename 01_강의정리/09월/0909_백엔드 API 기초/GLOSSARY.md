# GLOSSARY

백엔드 API 기초에서 자주 사용하는 용어를 빠르게 찾아볼 수 있도록 정리했습니다.

| 용어 | 설명 |
|---|---|
| API | 서로 다른 소프트웨어가 정해진 규칙으로 기능과 데이터를 주고받는 인터페이스 |
| Backend | 요청을 처리하고 업무 규칙, 데이터 저장, 외부 서비스 연동을 담당하는 서버 영역 |
| Client | API에 요청을 보내고 응답을 소비하는 프로그램 |
| Server | 요청을 받아 처리 결과를 응답하는 프로그램 또는 시스템 |
| Service | 업무 규칙과 핵심 처리를 담당하는 계층 또는 독립 기능 단위 |
| HTTP | 웹에서 요청과 응답을 교환하기 위한 application protocol |
| Request | Client가 server에 보내는 method, target, header, body로 구성된 메시지 |
| Response | Server가 처리 결과를 status, header, body로 전달하는 메시지 |
| Method | GET, POST, PUT, PATCH, DELETE처럼 요청 의도를 나타내는 값 |
| Header | 인증, 콘텐츠 형식, cache 등 메시지의 부가 정보를 담는 영역 |
| Body | 요청 데이터나 응답 결과가 담기는 메시지 본문 |
| Status Code | HTTP 처리 결과를 세 자리 숫자로 표현한 값 |
| Content-Type | 현재 메시지 body의 media type을 알리는 header |
| Accept | Client가 받을 수 있거나 선호하는 응답 media type을 알리는 header |
| JSON | Object와 array를 포함한 구조화 데이터를 표현하는 텍스트 형식 |
| Serialization | 프로그램 객체를 전송·저장 가능한 형식으로 바꾸는 과정 |
| Deserialization | 전송된 표현을 프로그램이 다룰 수 있는 값으로 복원하는 과정 |
| Schema | Field, type, 필수 여부, 제약 조건을 명시한 데이터 계약 |
| Validation | 입력이 형식, 타입, 업무 규칙을 만족하는지 확인하는 과정 |
| REST | 리소스와 HTTP 의미를 중심으로 API를 설계하는 architectural style |
| Resource | API가 식별하고 조회·변경하는 대상 |
| Endpoint | 특정 API 기능을 가리키는 method와 URL의 조합 |
| Path Parameter | URL path 안에서 특정 리소스를 식별하는 값 |
| Query Parameter | 필터, 정렬, 페이지 같은 조회 조건을 URL에 전달하는 값 |
| Handler | HTTP 요청을 받아 service를 호출하고 응답으로 변환하는 함수 |
| Middleware | 여러 요청에 공통으로 적용되는 인증, 로깅 등의 처리 계층 |
| Authentication | 요청 주체가 누구인지 확인하는 과정 |
| Authorization | 인증된 주체가 작업 권한을 가졌는지 확인하는 과정 |
| Idempotent | 같은 요청을 반복해도 의도된 최종 상태가 같은 성질 |
| Pagination | 큰 목록을 여러 페이지나 cursor 단위로 나누어 조회하는 방식 |
| Timeout | 작업을 더 기다리지 않고 실패로 처리하기까지의 제한 시간 |
| Contract Test | Client와 server가 합의한 요청·응답 형식을 지키는지 확인하는 테스트 |
| Correlation ID | Client와 server의 관련 로그를 하나의 요청 흐름으로 연결하는 식별자 |

## 연관 문서

- [API, Backend, Service의 역할](01-api-backend-and-service.md)
- [HTTP 요청과 응답 메시지](02-http-request-response.md)
- [JSON과 데이터 직렬화](03-json-serialization.md)
- [REST 리소스와 Endpoint 설계](04-rest-resource-design.md)
- [요청 처리 흐름과 입력 검증](05-request-lifecycle-validation.md)
- [API 클라이언트 호출과 테스트](06-api-client-testing.md)
- [전체 학습 개요](OVERVIEW.md)
