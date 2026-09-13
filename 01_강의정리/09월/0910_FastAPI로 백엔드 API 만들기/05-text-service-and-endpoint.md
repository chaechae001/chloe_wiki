# 텍스트 처리 Service와 Endpoint

> Endpoint는 HTTP 입출구를 담당하고, 텍스트 변환 규칙은 독립 함수로 분리하면 이해와 테스트가 쉬워집니다.

`service function` · `request model` · `response model` · `separation of concerns` · `Enum`

## 핵심요약

- Request model은 입력, response model은 출력 계약을 표현합니다.
- 변환 규칙은 endpoint 밖의 순수 함수로 분리할 수 있습니다.
- 지원 mode를 명시하면 잘못된 입력을 조기에 막을 수 있습니다.
- 계산 결과의 타입과 response schema를 일치시켜야 합니다.
- 작은 service 함수는 API 없이 빠르게 단위 테스트할 수 있습니다.

## 1. 입출력 모델 설계

```python
from enum import Enum
from pydantic import BaseModel, Field

class TransformMode(str, Enum):
    upper = "upper"
    lower = "lower"
    reverse = "reverse"
    length = "length"

class TransformRequest(BaseModel):
    text: str = Field(min_length=1, max_length=2000)
    mode: TransformMode = TransformMode.upper

class TransformResponse(BaseModel):
    result: str
    mode: TransformMode
```

Enum을 사용하면 지원하지 않는 mode를 schema 검증 단계에서 차단하고 자동 문서에도 가능한 값이 표시됩니다. 문자열을 직접 비교하는 설계도 가능하지만 허용 범위를 한곳에서 관리해야 합니다.

<details>
<summary>답</summary>

기본값은 client가 mode를 생략했을 때의 동작을 결정합니다. 편리하지만 암묵적인 동작이므로 자동 문서와 테스트에서 명확히 보여줘야 합니다.

</details>

## 2. 처리 로직 분리

```python
def transform_text(text: str, mode: TransformMode) -> str:
    operations = {
        TransformMode.upper: str.upper,
        TransformMode.lower: str.lower,
        TransformMode.reverse: lambda value: value[::-1],
        TransformMode.length: lambda value: str(len(value)),
    }
    return operations[mode](text)
```

이 함수는 HTTP, FastAPI, database에 의존하지 않습니다. 같은 입력에는 같은 결과를 내므로 mode별 결과와 경계값을 간단한 unit test로 검증할 수 있습니다.

<details>
<summary>답</summary>

길이 결과를 문자열로 반환한 이유는 response의 `result: str` 계약을 유지하기 위해서입니다. 숫자 결과를 그대로 쓰고 싶다면 `result` 타입을 union이나 별도 모델로 설계해야 합니다.

</details>

## 3. Endpoint에서 조립하기

```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post(
    "/transforms",
    response_model=TransformResponse,
    status_code=status.HTTP_200_OK,
)
def create_transform(payload: TransformRequest) -> TransformResponse:
    result = transform_text(payload.text, payload.mode)
    return TransformResponse(result=result, mode=payload.mode)
```

Endpoint는 검증된 요청을 받고 service를 호출한 뒤 response model에 맞게 반환합니다. 저장 없이 즉시 계산 결과를 주는 동작이라 여기서는 200을 사용했습니다.

## 코드로 보기 — Service 단위 테스트

```python
def test_transform_text_length() -> None:
    result = transform_text("API study", TransformMode.length)
    assert result == "9"

def test_transform_text_reverse() -> None:
    result = transform_text("abcd", TransformMode.reverse)
    assert result == "dcba"
```

### 코드 목적

HTTP 계층과 무관하게 핵심 텍스트 규칙을 검증합니다.

### 코드 흐름

1. 작은 입력과 mode를 준비합니다.
2. Service 함수를 직접 호출합니다.
3. 반환값을 기대 결과와 비교합니다.
4. Endpoint test에서는 연결과 schema만 추가 확인합니다.

### 실행 결과 해석

테스트가 통과하면 두 변환 규칙의 결과가 계약과 일치합니다. Unicode 글자 수나 공백 처리처럼 정의가 필요한 경계 조건은 별도 test로 추가합니다.

### 실무 연결

나중에 텍스트 처리 구현을 외부 AI service로 바꾸더라도 endpoint 계약과 service interface를 유지하면 변경 범위를 줄일 수 있습니다.

## 직접 해보기

1. Title case mode를 추가하세요.
2. 빈 문자열을 request model에서 거부하세요.
3. 단어 수 결과를 숫자로 제공하려면 response schema를 어떻게 바꿀지 설명하세요.

<details>
<summary>정답 보기</summary>

1. Enum 값과 `str.title` 연산을 mapping에 추가합니다.
2. `Field(min_length=1)`을 사용합니다.
3. 결과 타입을 `str | int`로 넓히거나 숫자 통계용 별도 field·response model을 두어 계약을 명시합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Request model vs response model | Client 입력 검증 vs server 출력 검증·필터링 |
| Endpoint vs service | HTTP/HTTP 계약 처리 vs 핵심 업무 규칙 |
| Default fallback vs explicit validation | 모르는 값도 임의 처리 vs 허용 목록 밖 입력을 오류로 반환 |

## 연결되는 개념

- 이전: [TestClient로 API 검증하기](04-testclient-api-testing.md)
- 다음: [응답 검증과 오류 처리](06-response-validation-and-errors.md)
- 함께 보면 좋은 키워드: `pure function`, `dependency injection`, `adapter`

## 셀프 체크

- [ ] Request와 response model의 역할을 구분할 수 있다.
- [ ] 처리 로직을 endpoint 밖으로 분리할 수 있다.
- [ ] Enum으로 허용 값을 제한할 수 있다.
- [ ] Service 함수의 unit test를 작성할 수 있다.
- [ ] 결과 타입과 response schema를 일치시킬 수 있다.

### 복습 질문 및 답변

**Q1. 처리 함수를 먼저 직접 테스트하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

HTTP 연결 문제와 핵심 규칙 문제를 분리해 빠르게 원인을 찾고, 작은 범위에서 다양한 입력을 검증할 수 있기 때문입니다.

</details>

**Q2. 지원하지 않는 mode를 원문 반환으로 처리하면 어떤 문제가 생기나요?**

<details>
<summary>답</summary>

Client의 오타가 성공처럼 보이고 잘못된 사용을 발견하기 어려워집니다. 명시적인 검증 오류가 계약을 더 선명하게 만듭니다.

</details>

**Q3. 텍스트 길이의 정의가 왜 중요할까요?**

<details>
<summary>답</summary>

Python의 `len` 결과와 사용자가 인식하는 글자 수가 결합 문자나 emoji에서 다를 수 있어 제품 요구에 맞는 기준을 정해야 합니다.

</details>

## 한 줄 정리

> 입력·출력 schema와 독립 service 함수를 조합하면 작은 FastAPI 기능도 명확하고 테스트 가능한 구조가 됩니다.
