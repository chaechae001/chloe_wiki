# OpenAPI와 자동 API 문서

> FastAPI의 자동 문서는 별도 문서가 아니라 실행 코드에서 생성되는 API 계약의 또 다른 표현입니다.

`OpenAPI` · `JSON Schema` · `Swagger UI` · `ReDoc` · `operationId`

## 핵심요약

- FastAPI는 path operation과 모델에서 OpenAPI schema를 생성합니다.
- 기본 Swagger UI는 `/docs`, ReDoc은 `/redoc`에서 확인합니다.
- 원시 schema는 기본적으로 `/openapi.json`에서 제공합니다.
- 앱과 endpoint 메타데이터가 문서의 가독성을 결정합니다.
- 자동 문서는 테스트를 돕지만 계약 검증을 대신하지는 않습니다.

## 1. 코드에서 문서가 만들어지는 흐름

```text
Decorator + type hints + Pydantic models + metadata
                    ↓
              OpenAPI schema
               ↙          ↘
        Swagger UI        ReDoc
```

`app.openapi()`를 호출하면 현재 애플리케이션의 schema dictionary를 확인할 수 있습니다. `paths`에는 등록된 path와 method, `components.schemas`에는 재사용되는 데이터 모델이 들어갑니다.

```python
schema = app.openapi()
for path, operations in schema["paths"].items():
    methods = ", ".join(operations).upper()
    print(f"{methods:8} {path}")
```

<details>
<summary>답</summary>

OpenAPI는 API 구조를 기계가 읽을 수 있게 기술하는 specification입니다. Swagger UI는 그 schema를 이용해 사람이 보고 요청까지 실행할 수 있는 화면을 제공합니다.

</details>

## 2. 문서 품질을 높이는 메타데이터

```python
app = FastAPI(
    title="Notes API",
    summary="짧은 메모를 관리하는 예제 API",
    version="1.0.0",
)

@app.get(
    "/notes/{note_id}",
    summary="노트 한 건 조회",
    tags=["notes"],
)
def read_note(note_id: int) -> dict:
    return {"id": note_id, "title": "sample"}
```

Title, summary, description, tags와 response model은 단순 장식이 아닙니다. Frontend 개발자와 client 생성 도구가 endpoint의 의도와 데이터 구조를 이해하는 근거가 됩니다.

<details>
<summary>답</summary>

앱의 `version`은 작성한 API 애플리케이션의 버전이며 OpenAPI specification 자체의 버전과는 별개입니다.

</details>

## 3. 문서 화면에서 확인할 것

- Method와 path가 의도대로 등록됐는가
- Path·query·body parameter의 필수 여부가 맞는가
- Request와 response schema가 실제 계약과 맞는가
- 성공·실패 상태 코드 설명이 충분한가
- 보안 scheme이 필요한 endpoint에 표시되는가

운영 환경에서 자동 문서 공개 여부는 보안 정책에 따라 결정합니다. `docs_url`, `redoc_url`, `openapi_url`을 바꾸거나 비활성화할 수 있지만, 문서를 숨기는 것 자체가 인증·인가를 대신하지는 않습니다.

## 코드로 보기 — schema를 테스트 대상으로 사용하기

```python
def test_openapi_contains_notes_path() -> None:
    schema = app.openapi()
    assert "/notes/{note_id}" in schema["paths"]
    assert "get" in schema["paths"]["/notes/{note_id}"]
```

### 코드 목적

핵심 endpoint가 OpenAPI 계약에 포함되는지 자동 확인합니다.

### 코드 흐름

1. 앱에서 schema를 생성합니다.
2. 필요한 path를 찾습니다.
3. 허용해야 할 method를 확인합니다.
4. 변경 시 테스트 실패로 계약 변화를 알립니다.

### 실행 결과 해석

Assertion이 통과하면 해당 path와 method가 schema에 존재합니다. 응답 내용까지 맞다는 뜻은 아니므로 API 호출 테스트도 필요합니다.

### 실무 연결

OpenAPI schema는 문서, client SDK 생성, API gateway 검증과 contract test의 공통 입력이 될 수 있습니다.

## 직접 해보기

1. 기본 Swagger UI와 원시 OpenAPI schema의 URL을 각각 적으세요.
2. `app.openapi()`에서 등록된 path 목록을 출력하세요.
3. 문서에 endpoint의 그룹과 짧은 설명을 추가해 보세요.

<details>
<summary>정답 보기</summary>

1. 기본값은 `/docs`와 `/openapi.json`입니다.
2. `for path in app.openapi()["paths"]: print(path)`처럼 순회합니다.
3. Decorator에 `tags=["그룹"]`, `summary="설명"`을 지정할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| OpenAPI vs Swagger UI | API schema specification vs schema를 보여주는 대화형 UI |
| `/docs` vs `/openapi.json` | 사람이 쓰는 화면 vs 기계가 읽는 원시 schema |
| 문서 노출 제한 vs API 보안 | 보조적인 노출 정책 vs 실제 인증·인가 통제 |

## 연결되는 개념

- 이전: [요청 파라미터와 Pydantic 모델](02-request-parameters-and-pydantic.md)
- 다음: [TestClient로 API 검증하기](04-testclient-api-testing.md)
- 함께 보면 좋은 키워드: `contract`, `client generation`, `schema evolution`

## 셀프 체크

- [ ] OpenAPI와 Swagger UI를 구분할 수 있다.
- [ ] `app.openapi()`의 `paths`를 탐색할 수 있다.
- [ ] 자동 문서에 모델 정보가 반영되는 과정을 설명할 수 있다.
- [ ] 앱과 endpoint 메타데이터를 지정할 수 있다.
- [ ] 문서 확인과 자동 테스트의 차이를 이해한다.

### 복습 질문 및 답변

**Q1. 코드 변경 뒤 문서가 함께 바뀌는 이유는 무엇인가요?**

<details>
<summary>답</summary>

FastAPI가 실행 코드의 route, type hint, model과 metadata에서 OpenAPI schema를 생성하기 때문입니다.

</details>

**Q2. `/docs`에서 성공한 호출만 확인하면 충분한가요?**

<details>
<summary>답</summary>

아닙니다. 필수값 누락, 타입 오류, 권한 실패 같은 실패 계약과 반복 가능한 자동 테스트도 함께 확인해야 합니다.

</details>

**Q3. 자동 문서를 끄면 endpoint도 사라지나요?**

<details>
<summary>답</summary>

문서 URL과 OpenAPI URL 설정만 비활성화한다면 일반 endpoint는 그대로 동작합니다.

</details>

## 한 줄 정리

> FastAPI는 코드의 요청·응답 선언을 OpenAPI로 바꾸고, 그 schema가 문서와 자동화의 공통 계약이 됩니다.
