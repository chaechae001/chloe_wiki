# 응답 검증과 오류 처리

> 요청 검증만큼 중요한 것이 서버가 약속한 형태로 응답하는지 확인하는 일입니다.

`response_model` · `ResponseValidationError` · `HTTPException` · `error contract` · `status code`

## 핵심요약

- `response_model`은 문서화뿐 아니라 출력 검증과 filtering에 사용됩니다.
- 반환 field가 schema와 다르면 client 오류가 아니라 server 구현 오류입니다.
- 예상 가능한 업무 오류는 명시적인 status와 메시지로 변환합니다.
- Response field 이름은 구현과 schema에서 정확히 일치해야 합니다.
- 성공과 실패 응답을 모두 테스트해야 계약이 완성됩니다.

## 1. Response model의 역할

```python
class TransformDetail(BaseModel):
    result: str
    mode: TransformMode
    source_length: int
    result_length: int

@app.post("/transform-details", response_model=TransformDetail)
def create_transform_detail(payload: TransformRequest) -> dict:
    result = transform_text(payload.text, payload.mode)
    return {
        "result": result,
        "mode": payload.mode,
        "source_length": len(payload.text),
        "result_length": len(result),
    }
```

Response model은 OpenAPI schema를 만들고, 반환 데이터를 검증·직렬화하며, 선언되지 않은 field를 걸러낼 수 있습니다. 따라서 field 철자 하나가 달라도 server error가 될 수 있습니다.

<details>
<summary>답</summary>

Request validation 실패는 client가 계약에 맞지 않는 값을 보낸 상황이고, response validation 실패는 server 코드가 자신이 선언한 계약을 지키지 못한 상황입니다.

</details>

## 2. 예측 가능한 오류 표현

Enum으로 허용 값을 제한하지 않고 업무 로직에서 판단해야 한다면 `HTTPException`으로 오류를 명시할 수 있습니다.

```python
from fastapi import HTTPException

SUPPORTED_MODES = {"upper", "lower"}

def validate_mode(mode: str) -> None:
    if mode not in SUPPORTED_MODES:
        raise HTTPException(
            status_code=400,
            detail={"code": "UNSUPPORTED_MODE", "message": "지원하지 않는 처리 방식입니다."},
        )
```

400은 요청의 의미가 허용 규칙에 맞지 않을 때 사용할 수 있습니다. Pydantic schema 단계에서 허용 값 검증이 실패하면 FastAPI의 기본 동작에서는 422가 될 수 있으므로 어느 계층이 규칙을 담당할지 일관되게 정합니다.

<details>
<summary>답</summary>

모든 예외를 200 응답 body 안에 넣으면 HTTP client, monitoring, retry 정책이 성공과 실패를 구분하기 어려워집니다. 상황에 맞는 status code와 안정적인 오류 code를 함께 사용합니다.

</details>

## 3. 오류를 재현하는 테스트

```python
def test_transform_response_contract() -> None:
    response = client.post(
        "/transform-details",
        json={"text": "Hello", "mode": "upper"},
    )
    assert response.status_code == 200
    assert response.json() == {
        "result": "HELLO",
        "mode": "upper",
        "source_length": 5,
        "result_length": 5,
    }
```

테스트는 field 이름, 타입, 계산 의미까지 확인합니다. Response model의 이름과 반환 dictionary의 key가 다르면 TestClient는 기본 설정에서 server exception을 test에 드러내므로 구현 오류를 바로 찾을 수 있습니다.

## 코드로 보기 — 실패 계약 검사

```python
def test_rejects_unknown_mode() -> None:
    response = client.post(
        "/legacy-transforms",
        json={"text": "Hello", "mode": "unknown"},
    )
    assert response.status_code == 400
    assert response.json()["detail"]["code"] == "UNSUPPORTED_MODE"
```

### 코드 목적

지원하지 않는 값이 조용히 성공하지 않고 명시적 오류로 반환되는지 확인합니다.

### 코드 흐름

1. Schema 문법상 유효하지만 업무 규칙상 잘못된 값을 보냅니다.
2. 오류 status를 확인합니다.
3. Client가 분기할 안정적인 오류 code를 확인합니다.
4. 내부 stack trace가 body에 노출되지 않는지 봅니다.

### 실행 결과 해석

400과 `UNSUPPORTED_MODE`가 확인되면 client가 사람용 message에 의존하지 않고 오류 종류를 처리할 수 있습니다.

### 실무 연결

일관된 오류 계약은 frontend 안내, monitoring 분류, support 문의와 API version 관리에 사용됩니다.

## 직접 해보기

1. Response model의 필수 field 하나를 반환하지 않으면 누구의 오류인지 설명하세요.
2. 지원하지 않는 mode를 400으로 반환하는 코드를 작성하세요.
3. 성공 응답에 내부용 secret field가 섞여 있을 때 response model이 줄 수 있는 보호를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 선언한 출력 계약을 server가 지키지 못한 구현 오류입니다.
2. 허용 목록을 확인한 뒤 `raise HTTPException(status_code=400, detail=...)`을 사용합니다.
3. 선언되지 않은 field를 출력에서 filtering해 의도하지 않은 데이터 노출을 줄일 수 있으며, 별도 보안 검토도 함께 필요합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Request validation vs response validation | Client 입력 문제 vs server 반환 계약 문제 |
| 400 vs 422 | 업무 의미·일반 요청 오류로 쓰는 400 vs schema 해석 후 validation 실패에 흔히 쓰는 422 |
| Error message vs error code | 사람이 읽는 설명 vs client가 안정적으로 분기할 식별자 |

## 연결되는 개념

- 이전: [텍스트 처리 Service와 Endpoint](05-text-service-and-endpoint.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)
- 함께 보면 좋은 키워드: `exception handler`, `observability`, `schema versioning`

## 셀프 체크

- [ ] `response_model`의 세 가지 역할을 설명할 수 있다.
- [ ] Request와 response validation 실패를 구분할 수 있다.
- [ ] 업무 오류에 `HTTPException`을 적용할 수 있다.
- [ ] 오류 message와 code를 구분해 설계할 수 있다.
- [ ] 성공·실패 response 계약 테스트를 작성할 수 있다.

### 복습 질문 및 답변

**Q1. 반환 dictionary에 response model보다 field가 많으면 어떻게 되나요?**

<details>
<summary>답</summary>

일반적으로 response model에 선언되지 않은 field는 filtering됩니다. 이 기능은 출력 계약 유지와 민감 정보 노출 방지에 도움이 됩니다.

</details>

**Q2. Field 이름의 오타가 왜 단순 표시 문제가 아닌가요?**

<details>
<summary>답</summary>

Schema의 필수 field를 충족하지 못하거나 다른 field를 반환해 response validation이 실패하고 5xx server error로 이어질 수 있습니다.

</details>

**Q3. 모든 허용 값 검증을 Pydantic Enum으로 옮겨야 하나요?**

<details>
<summary>답</summary>

고정된 입력 집합은 Enum이 편리하지만 권한·현재 상태처럼 실행 시점의 외부 정보가 필요한 규칙은 service에서 검증하는 편이 자연스럽습니다.

</details>

## 한 줄 정리

> Response model과 오류 계약은 서버의 출력도 검증 가능한 약속으로 만들며, field 불일치와 조용한 실패를 조기에 드러냅니다.
