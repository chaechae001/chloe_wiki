# 계약 테스트와 운영 점검

> 실제 모델 호출과 웹 API 계약을 분리해 테스트하면 빠르고 재현 가능한 검증이 가능합니다.

`TestClient` · `mock` · `contract test` · `health check` · `OpenAPI`

## 핵심요약

- Swagger UI는 탐색용이고 자동화 test는 회귀 방지용입니다.
- 단위·endpoint test에서는 LLM 호출을 가짜 service로 대체합니다.
- 성공뿐 아니라 공백, 길이 초과, timeout, rate limit을 검증합니다.
- Health endpoint는 프로세스 상태와 외부 의존성 상태를 구분합니다.
- 실제 공급자 호출 test는 별도 통합 환경에서 최소화합니다.

## 1. 테스트 피라미드

```text
많음  service unit test
      FastAPI contract test with fake provider
적음  real provider integration test
```

실제 LLM 답변 문장을 정확히 비교하면 비결정성과 모델 변경 때문에 test가 깨집니다. 대신 HTTP status, schema, 비어 있지 않은 reply, 오류 code 같은 계약을 검증합니다.

<details>
<summary>답</summary>

생성 내용의 품질 평가는 별도 평가 dataset과 rubric으로 다루고, 웹 API test는 구조와 실패 동작에 집중합니다.

</details>

## 2. Endpoint 계약 예시

```python
from fastapi.testclient import TestClient

def test_blank_message_is_rejected(client: TestClient):
    response = client.post("/chat", json={"message": "   "})
    assert response.status_code == 400

def test_wrong_type_is_rejected(client: TestClient):
    response = client.post("/chat", json={"message": ["wrong"]})
    assert response.status_code == 422
```

성공 test에서는 가짜 service가 고정 답변을 반환하도록 주입해 네트워크와 비용을 제거합니다.

## 3. 운영 체크리스트

- `/health/live`: 프로세스가 요청을 받을 수 있는지
- `/health/ready`: 필수 내부 의존성을 사용할 수 있는지
- 요청 크기·속도 제한
- Timeout, retry, 동시성 제한
- 구조화 로그와 민감정보 제거
- 배포 전 secret scan과 OpenAPI schema diff

## 코드로 보기 — 실패 주입

### 코드 목적

공급자 장애 상황을 실제 장애 없이 재현합니다.

### 코드 흐름

1. 가짜 service가 timeout 예외를 냅니다.
2. Endpoint가 오류를 매핑합니다.
3. Test가 status와 공개 detail을 확인합니다.

### 실행 결과 해석

외부 API가 없어도 서비스의 실패 계약을 반복 검증할 수 있습니다.

### 실무 연결

CI에서는 secret scan, lint, contract test를 실행하고 실제 호출 test는 권한이 제한된 환경으로 분리합니다.

## 직접 해보기

1. 가짜 성공 service로 200 schema를 검사하세요.
2. Timeout 예외로 504를 검사하세요.
3. OpenAPI에 `/chat`의 request·response model이 있는지 확인하세요.

<details>
<summary>정답 보기</summary>

Dependency override나 monkeypatch로 provider 함수를 바꾸고 `status_code`, JSON field, 오류 detail을 assertion합니다. `/openapi.json`의 `paths`와 schema도 검증할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Swagger UI vs automated test | 수동 탐색 도구와 반복 가능한 회귀 검증입니다. |
| Liveness vs readiness | 프로세스 생존과 요청 처리 준비 상태입니다. |
| Mock test vs integration test | 내 계약만 검증하는 test와 실제 외부 연결을 포함하는 test입니다. |

## 연결되는 개념

- 이전: [오류 처리·재시도·관측성](05-errors-retries-and-observability.md)
- 함께 보기: [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 생성 문장 대신 API 계약을 검증한다.
- [ ] 가짜 service로 외부 호출을 제거한다.
- [ ] 정상·검증·외부 장애 case를 테스트한다.
- [ ] Liveness와 readiness를 구분한다.
- [ ] CI에서 secret scan을 수행한다.

### 복습 질문 및 답변

**Q1. `/docs`에서 성공했으면 test가 필요 없나요?**

<details>
<summary>답</summary>

수동 확인은 반복성과 회귀 탐지가 부족하므로 자동화 test가 필요합니다.

</details>

**Q2. Test에서 실제 모델 답변 전체를 비교해야 하나요?**

<details>
<summary>답</summary>

웹 계약 test에서는 고정된 가짜 응답을 쓰고, 품질 평가는 별도 평가로 분리하는 편이 안정적입니다.

</details>

**Q3. Readiness에서 매번 생성 요청을 보내야 하나요?**

<details>
<summary>답</summary>

비용과 장애 증폭을 피하도록 값싼 내부 점검을 사용하고 실제 공급자 probe는 별도 정책으로 제한합니다.

</details>

## 한 줄 정리

> 가짜 공급자로 HTTP 계약을 반복 검증하고 실제 호출은 제한된 통합 test로 분리합니다.
