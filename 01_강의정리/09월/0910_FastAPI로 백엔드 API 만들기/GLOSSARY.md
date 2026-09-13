# 용어집

FastAPI 기반 API 서버를 만들고 검증할 때 자주 만나는 용어를 쉬운 말로 정리했습니다.

## 서버와 Routing

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| FastAPI | Python type hint를 활용해 API를 만들고 검증·문서화를 연결하는 web framework | [앱 구조](01-fastapi-app-and-server.md) | Pydantic, OpenAPI |
| ASGI | 비동기 Python web application과 server 사이의 표준 interface | [앱 구조](01-fastapi-app-and-server.md) | Uvicorn, event loop |
| Uvicorn | ASGI application을 실행해 network request를 전달하는 server | [앱 구조](01-fastapi-app-and-server.md) | process, reload |
| Path operation | HTTP method와 path에 연결된 요청 처리 규칙 | [앱 구조](01-fastapi-app-and-server.md) | endpoint, decorator |
| Decorator | 함수를 route에 등록하는 `@app.get(...)` 같은 Python 문법 | [앱 구조](01-fastapi-app-and-server.md) | routing |
| Reload | 개발 중 파일 변경을 감지해 server를 다시 시작하는 기능 | [앱 구조](01-fastapi-app-and-server.md) | development server |

## 요청과 데이터 모델

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Path parameter | URL 경로 안에서 특정 대상을 식별하는 값 | [요청 모델](02-request-parameters-and-pydantic.md) | route |
| Query parameter | 검색, 필터, 페이지 같은 선택 조건을 URL 뒤에 전달하는 값 | [요청 모델](02-request-parameters-and-pydantic.md) | default value |
| Request body | Client가 server에 보내는 구조화된 본문 데이터 | [요청 모델](02-request-parameters-and-pydantic.md) | JSON |
| Pydantic | Python type을 기반으로 데이터를 parsing하고 검증하는 library | [요청 모델](02-request-parameters-and-pydantic.md) | BaseModel, Field |
| Schema | Field 이름, 타입, 필수 여부와 제약을 표현한 데이터 계약 | [요청 모델](02-request-parameters-and-pydantic.md) | validation |
| 422 | 요청을 읽었지만 선언된 입력 검증을 통과하지 못했을 때 FastAPI에서 흔히 보는 status | [요청 모델](02-request-parameters-and-pydantic.md) | validation error |
| Enum | 선택 가능한 값을 제한하는 열거형 | [텍스트 Service](05-text-service-and-endpoint.md) | allowed values |

## 문서와 테스트

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| OpenAPI | API path, parameter, request와 response schema를 기술하는 표준 | [자동 문서](03-openapi-and-automatic-docs.md) | JSON Schema |
| Swagger UI | OpenAPI schema를 사람이 탐색하고 호출할 수 있게 보여주는 화면 | [자동 문서](03-openapi-and-automatic-docs.md) | `/docs` |
| ReDoc | OpenAPI 기반의 또 다른 API 문서 화면 | [자동 문서](03-openapi-and-automatic-docs.md) | `/redoc` |
| TestClient | 별도 socket server 없이 FastAPI application을 호출하는 test client | [API 테스트](04-testclient-api-testing.md) | HTTPX, pytest |
| Assertion | 실제 결과가 기대 조건을 만족하는지 자동 판정하는 검사 | [API 테스트](04-testclient-api-testing.md) | regression test |
| Hidden state | Notebook의 과거 변수와 실행 순서가 현재 결과에 영향을 주는 상태 | [API 테스트](04-testclient-api-testing.md) | reproducibility |

## 응답과 오류

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| Response model | Server가 반환할 데이터의 타입과 공개 field를 정의하는 모델 | [응답과 오류](06-response-validation-and-errors.md) | filtering |
| Response validation | Server 반환값이 선언한 response schema와 맞는지 확인하는 과정 | [응답과 오류](06-response-validation-and-errors.md) | server error |
| HTTPException | 의도한 HTTP status와 오류 detail을 반환하기 위한 FastAPI exception | [응답과 오류](06-response-validation-and-errors.md) | error handler |
| Error code | Client가 오류 종류를 안정적으로 구분할 수 있는 식별자 | [응답과 오류](06-response-validation-and-errors.md) | message |
| Service function | HTTP 세부사항과 분리된 핵심 처리 로직 | [텍스트 Service](05-text-service-and-endpoint.md) | unit test |

## 함께 보기

- [전체 학습 개요](OVERVIEW.md)
