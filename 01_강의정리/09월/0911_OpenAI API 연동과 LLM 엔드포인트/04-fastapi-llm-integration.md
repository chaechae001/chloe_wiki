# FastAPI와 LLM 연결

> FastAPI endpoint는 입력을 검증하고 서비스 함수에 위임한 뒤 안정적인 응답 모델로 결과를 반환합니다.

`FastAPI` · `Pydantic` · `dependency` · `service layer` · `async boundary`

## 핵심요약

- SDK 호출을 endpoint 본문과 분리하면 테스트와 교체가 쉬워집니다.
- Pydantic으로 빈 문자열, 길이, 선택값 범위를 요청 경계에서 검증합니다.
- 서버가 system instruction과 model 선택 권한을 유지합니다.
- 동기 SDK 호출을 `async def`에서 무심코 실행하면 event loop를 막을 수 있습니다.
- 응답 모델로 외부에 노출할 field를 제한합니다.

## 1. 요청과 응답 모델

```python
from pydantic import BaseModel, Field

class ChatRequest(BaseModel):
    message: str = Field(min_length=1, max_length=4000)

class ChatResponse(BaseModel):
    reply: str
    request_id: str
```

길이 제한은 비용·지연·악용을 줄이는 첫 경계입니다. 공백만 있는 값은 validator 또는 endpoint 앞단에서 별도로 거절합니다.

<details>
<summary>답</summary>

타입이 문자열이라는 사실만으로 의미 있는 입력임이 보장되지 않으므로 trim 후 빈 값도 검사해야 합니다.

</details>

## 2. 서비스 함수 분리

```python
def generate_reply(message: str) -> str:
    response = client.responses.create(
        model=os.environ["OPENAI_MODEL"],
        instructions="정확하고 간결하게 답하세요.",
        input=message,
    )
    return response.output_text
```

## 3. Endpoint 구성

```python
from uuid import uuid4
from fastapi import FastAPI, HTTPException

app = FastAPI(title="LLM Backend")

@app.post("/chat", response_model=ChatResponse)
def chat(req: ChatRequest) -> ChatResponse:
    message = req.message.strip()
    if not message:
        raise HTTPException(400, "message must not be blank")
    reply = generate_reply(message)
    return ChatResponse(reply=reply, request_id=str(uuid4()))
```

## 코드로 보기 — 요청 경계

### 코드 목적

웹 규칙과 외부 모델 호출 책임을 분리합니다.

### 코드 흐름

1. Pydantic이 기본 형식과 길이를 검증합니다.
2. Endpoint가 의미 검증을 수행합니다.
3. Service가 LLM을 호출합니다.
4. Response model이 공개 field를 제한합니다.

### 실행 결과 해석

형식 오류는 422, 명시한 의미 오류는 400, 성공은 정의된 JSON schema로 반환됩니다.

### 실무 연결

Dependency injection으로 가짜 service를 주입하면 실제 비용 없이 endpoint를 테스트할 수 있습니다.

## 직접 해보기

1. 메시지 최대 길이를 1000자로 제한하세요.
2. 공백 입력을 400으로 거절하세요.
3. Service를 가짜 함수로 바꿔 endpoint test를 작성하세요.

<details>
<summary>정답 보기</summary>

`Field(max_length=1000)`과 `if not req.message.strip()`을 사용합니다. 테스트에서는 고정 문자열을 반환하는 dependency나 monkeypatch를 사용할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| 400 vs 422 | 의미상 거절과 schema validation 실패입니다. |
| endpoint vs service | HTTP 계약 처리와 모델 공급자 호출 책임입니다. |
| sync vs async SDK | 실행 방식이 다르며 event loop에 맞게 선택해야 합니다. |

## 연결되는 개념

- 이전: [입력·출력과 대화 상태](03-input-output-and-conversation-state.md)
- 다음: [오류 처리와 재시도](05-errors-retries-and-observability.md)

## 셀프 체크

- [ ] Request·response model을 정의한다.
- [ ] 공백과 길이를 검증한다.
- [ ] Endpoint와 service 책임을 구분한다.
- [ ] System instruction을 서버가 관리한다.
- [ ] 동기 호출의 event loop 영향을 안다.

### 복습 질문 및 답변

**Q1. 사용자가 system prompt를 자유롭게 보내게 해도 되나요?**

<details>
<summary>답</summary>

제품 정책을 우회할 수 있으므로 일반 사용자에게는 서버가 정한 지침을 적용하고 허용된 옵션만 노출합니다.

</details>

**Q2. 왜 response model이 필요한가요?**

<details>
<summary>답</summary>

문서화와 검증을 제공하고 의도하지 않은 내부 field가 응답에 포함되는 것을 막습니다.

</details>

**Q3. 개발 서버의 reload를 운영에 써도 되나요?**

<details>
<summary>답</summary>

Reload는 개발 편의 기능입니다. 운영에서는 process·배포 환경에 맞는 실행 및 복구 정책을 사용합니다.

</details>

## 한 줄 정리

> FastAPI는 검증과 HTTP 계약을 맡고, LLM 호출은 교체 가능한 service 경계로 분리합니다.
