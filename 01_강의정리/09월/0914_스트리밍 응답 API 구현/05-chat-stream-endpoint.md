# chat stream endpoint 설계

> chat endpoint는 입력 검증, 서버 정책, 모델 stream 변환, 안전한 응답 형식을 한 경계에서 조합합니다.

`Pydantic` · `request schema` · `server policy` · `endpoint` · `sentinel`

## 핵심요약

- 요청 모델은 빈 값과 길이·옵션 범위를 먼저 검증합니다.
- 모델 선택과 system instruction은 서버 정책으로 관리합니다.
- endpoint는 stream source를 text 또는 SSE로 변환해 반환합니다.
- HTTP status는 시작 전 실패에 가장 유용하고, 시작 후 실패는 stream 내 신호가 필요할 수 있습니다.
- 내부 provider 객체를 그대로 공개하지 않습니다.

## 1. 요청 모델

```python
from pydantic import BaseModel, Field

class ChatRequest(BaseModel):
    message: str = Field(min_length=1, max_length=4000)
    temperature: float = Field(default=0.3, ge=0.0, le=2.0)
```

공백만 있는 문자열은 type·길이만으로 충분히 막히지 않을 수 있으므로 `strip()` 기반 의미 검증도 추가합니다.

<details>
<summary>답</summary>

사용자 요청에 임의 system prompt나 모델 ID를 그대로 허용하면 정책 우회, 비용·품질 변동이 생길 수 있어 허용 목록을 두는 편이 안전합니다.

</details>

## 2. endpoint 뼈대

```python
from fastapi import HTTPException
from fastapi.responses import StreamingResponse

@app.post("/chat/stream")
async def chat_stream(req: ChatRequest):
    if not req.message.strip():
        raise HTTPException(400, "message must not be blank")

    async def generate():
        async for piece in model_text_stream(req.message):
            yield piece

    return StreamingResponse(generate(), media_type="text/plain")
```

모델 호출은 `model_text_stream()` 같은 service 함수로 분리하면 실제 provider 없이 endpoint contract를 테스트할 수 있습니다.

## 3. 종료 계약

텍스트 stream에서 `[DONE]`, `[EMPTY]`, `[ERROR]` 같은 sentinel을 섞으면 빠른 실습에는 편하지만 사용자가 생성한 텍스트와 충돌할 수 있습니다. 운영 API는 SSE event 이름 또는 JSON line schema처럼 명시적인 protocol을 선택하는 편이 안전합니다.

## 직접 해보기

1. 공백 message를 400으로 거절하세요.
2. 고정된 service stream을 주입해 endpoint test를 작성하세요.
3. text sentinel 대신 SSE event schema를 설계하세요.

<details>
<summary>정답 보기</summary>

`if not req.message.strip()`로 의미 검증을 하고, fake generator로 `data: {"type":"done"}\n\n` 같은 완료 event를 검증할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| 400 vs 422 | 의미상 빈 입력과 schema validation 실패입니다. |
| endpoint vs stream service | HTTP 계약과 provider stream 변환 책임입니다. |
| text sentinel vs SSE event | 문자열 약속과 구조화된 이벤트 protocol입니다. |

## 연결되는 개념

- 이전: [FastAPI StreamingResponse](04-fastapi-streamingresponse.md)
- 다음: [오류·취소·결과 점검](06-stream-errors-cancellation-testing.md)

## 셀프 체크

- [ ] request schema를 정의한다.
- [ ] 공백 입력을 검사한다.
- [ ] 모델 정책을 서버에 둔다.
- [ ] endpoint와 service를 분리한다.
- [ ] 시작 후 오류의 전달 방식을 설계한다.

### 복습 질문 및 답변

**Q1. stream이 시작한 뒤 HTTP status를 500으로 바꿀 수 있나요?**

<details>
<summary>답</summary>

대개 header가 이미 전송됐으므로 어렵습니다. body event나 연결 종료를 포함한 protocol을 미리 정해야 합니다.

</details>

**Q2. 왜 model ID를 요청 body에 그대로 두지 않나요?**

<details>
<summary>답</summary>

허용되지 않은 모델 사용, 비용 예측 실패, 정책 일관성 저하를 막기 위해 서버가 선택권을 가져야 합니다.

</details>

**Q3. completion 객체 전체를 반환해도 되나요?**

<details>
<summary>답</summary>

provider 내부 구조가 공개 계약이 되어 변경과 정보 노출 위험이 커지므로 필요한 data만 변환합니다.

</details>

## 한 줄 정리

> chat stream은 요청 검증과 서버 정책을 지키며 provider 이벤트를 서비스 protocol로 번역합니다.
