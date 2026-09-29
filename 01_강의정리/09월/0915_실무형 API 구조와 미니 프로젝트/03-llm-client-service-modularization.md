# LLM Client와 Service 모듈화

> Client는 OpenAI SDK를 서비스 언어로 변환하고, Service는 프롬프트·오류·캐시 같은 업무 정책을 조립합니다.

`AsyncOpenAI` · `Responses API` · `async iterator` · `service boundary` · `provider adapter`

## 핵심요약

- Client만 OpenAI SDK를 import하도록 경계를 좁힙니다.
- 새 OpenAI 직접 통합은 Responses API를 기본으로 설계합니다.
- Client는 provider event를 text 조각이나 정규화된 결과로 바꿉니다.
- Service는 system instruction, 사용자 입력, 캐시와 완료 규칙을 관리합니다.
- Fake Client를 주입해 실제 비용 없이 Service를 테스트합니다.

## 1. Client의 역할

```python
import os
from collections.abc import AsyncIterator
from openai import AsyncOpenAI

client = AsyncOpenAI(api_key=os.environ["OPENAI_API_KEY"])

async def stream_text(user_input: str) -> AsyncIterator[str]:
    stream = await client.responses.create(
        model=os.environ["OPENAI_MODEL"],
        input=user_input,
        stream=True,
    )
    async for event in stream:
        if event.type == "response.output_text.delta":
            yield event.delta
```

SDK event와 model ID를 Router까지 끌어올리지 않고 Client에서 정규화합니다. 교육용 호환 환경이 Chat Completions를 요구한다면 같은 interface를 구현하는 별도 adapter로 둡니다.

<details>
<summary>답</summary>

Service가 SDK 객체 구조를 알지 않으면 Provider나 API 방식이 바뀌어도 Client 구현만 교체할 수 있습니다.

</details>

## 2. Service의 역할

```python
from collections.abc import AsyncIterator

async def answer_stream(message: str) -> AsyncIterator[str]:
    produced = False
    async for piece in stream_text(message):
        produced = True
        yield piece
    if not produced:
        raise EmptyModelResponse()
```

운영 환경에서는 문자열 `[ERROR]`을 생성 본문에 섞기보다 SSE event 또는 구조화된 protocol을 정의합니다. Service는 HTTP status가 아니라 도메인 상태를 표현하고 Router가 전송 형식으로 변환합니다.

## 3. Dependency 주입

함수 매개변수나 객체 생성자를 통해 Client interface를 넘기면 test에서 고정 조각을 내는 fake를 사용할 수 있습니다. 전역 객체에 강하게 묶인 monkeypatch보다 의존성이 명시적입니다.

## 직접 해보기

1. `stream_text()`가 문자열만 yield하도록 만드세요.
2. 빈 stream을 도메인 예외로 바꾸세요.
3. `['안', '녕']`을 내는 Fake Client로 Service를 테스트하세요.

<details>
<summary>정답 보기</summary>

Client에서 event type을 검사해 delta만 방출하고, Service는 `produced` flag로 빈 결과를 감지합니다. Fake Client를 주입하면 네트워크 없이 순서와 완료 규칙을 검증할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Client vs Service | Provider 통신과 제품 정책입니다. |
| SDK event vs service event | 외부 형식과 공개 API가 약속한 형식입니다. |
| Global client vs dependency | 암묵적 공유 객체와 교체 가능한 명시적 의존성입니다. |

## 연결되는 개념

- 이전: [설정과 Schema 설계](02-configuration-and-schemas.md)
- 다음: [System Prompt와 Template](04-system-prompts-and-templates.md)

## 셀프 체크

- [ ] SDK import를 Client에 한정한다.
- [ ] Provider event를 정규화한다.
- [ ] Service에서 업무 정책을 조립한다.
- [ ] 빈 결과와 오류를 구분한다.
- [ ] Fake Client로 테스트한다.

### 복습 질문 및 답변

**Q1. Service가 `StreamingResponse`를 반환해도 되나요?**

<details>
<summary>답</summary>

그렇게 하면 FastAPI에 결합됩니다. Service는 async iterator를 반환하고 Router가 HTTP 응답으로 감싸는 편이 낫습니다.

</details>

**Q2. 왜 Client가 문자열만 내보내나요?**

<details>
<summary>답</summary>

Service가 provider별 chunk·event 구조에 의존하지 않도록 필요한 공통 형태로 축소하기 위해서입니다.

</details>

**Q3. 실제 API를 호출하지 않고 무엇을 검증할 수 있나요?**

<details>
<summary>답</summary>

조각 순서, 빈 결과, 취소, 캐시 저장, 완료 event 같은 Service contract를 검증할 수 있습니다.

</details>

## 한 줄 정리

> Client는 외부 SDK를 번역하고 Service는 제품 동작을 결정합니다.
