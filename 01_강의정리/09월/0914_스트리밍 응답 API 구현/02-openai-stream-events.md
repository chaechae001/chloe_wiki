# OpenAI 스트림 이벤트 처리

> OpenAI Responses API 스트림은 SSE로 전달되는 타입 있는 이벤트에서 필요한 텍스트 delta와 완료·오류 신호를 골라 처리합니다.

`Responses API` · `SSE` · `response.output_text.delta` · `response.completed` · `error`

## 핵심요약

- 새 OpenAI 통합은 Responses API의 스트리밍을 기본으로 봅니다.
- `stream=True` 호출 결과는 이벤트를 순회하는 stream입니다.
- 텍스트는 `response.output_text.delta` 이벤트의 `delta`에서 얻습니다.
- 완료와 오류도 별도 이벤트로 처리합니다.
- 기존 Chat Completions 호환 서비스는 `choices[].delta.content` 구조일 수 있습니다.

## 1. Responses API 예시

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
stream = client.responses.create(
    model=os.environ["OPENAI_MODEL"],
    input="스트리밍을 한 문장으로 설명해줘.",
    stream=True,
)

for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
    elif event.type == "error":
        raise RuntimeError(event.message)
```

공식 가이드는 Responses API에 의미 있는 이벤트 유형을 제공하며, 텍스트 처리에는 `response.output_text.delta`, 완료 확인에는 `response.completed`를 사용하도록 설명합니다.

<details>
<summary>답</summary>

텍스트가 아닌 생성 lifecycle 이벤트도 오므로 모든 event를 텍스트로 취급하지 않고 `type`을 먼저 검사합니다.

</details>

## 2. Chat Completions 호환 형식

기존 교육 코드와 호환 게이트웨이는 `stream=True`로 받은 chunk에서 `choices[0].delta.content`를 읽을 수 있습니다. 빈 `choices`, role만 든 delta, 마지막 빈 chunk를 안전하게 건너뛰어야 합니다.

```python
for chunk in stream:
    if not chunk.choices:
        continue
    text = getattr(chunk.choices[0].delta, "content", None)
    if text:
        yield text
```

## 3. 조절과 안전

스트리밍 출력은 부분적으로 공개되므로, 완성본 기준 moderation이나 JSON 검증만으로는 늦을 수 있습니다. 출력 제한, 취소, UI 경고와 정책에 맞는 moderation 설계를 함께 고려합니다.

## 직접 해보기

1. delta 이벤트만 출력하세요.
2. 완료 이벤트를 만나면 완료 시각을 기록하세요.
3. 빈 chunk가 있어도 실패하지 않는 호환 루프를 작성하세요.

<details>
<summary>정답 보기</summary>

event의 `type`을 분기하고, 호환 루프에서는 `choices`와 `content`를 각각 확인합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Responses event vs Chat chunk | 이벤트 타입 중심 형식과 `choices[].delta` 중심 형식입니다. |
| delta vs 완성 텍스트 | 이번 조각과 전체 결과입니다. |
| 완료 event vs 연결 종료 | 의미 있는 생성 완료 신호와 네트워크 연결 종료는 다를 수 있습니다. |

## 연결되는 개념

- 이전: [스트리밍 응답의 원리](01-streaming-fundamentals.md)
- 다음: [async generator 파이프라인](03-async-generator-pipelines.md)

## 셀프 체크

- [ ] `stream=True`의 효과를 설명한다.
- [ ] 텍스트 delta 이벤트를 식별한다.
- [ ] 완료와 오류를 구분한다.
- [ ] 빈 호환 chunk를 처리한다.
- [ ] 스트리밍 moderation의 제약을 안다.

### 복습 질문 및 답변

**Q1. 모든 stream event에 텍스트가 있나요?**

<details>
<summary>답</summary>

아닙니다. 생성 시작, 완료, 오류처럼 lifecycle 정보를 가진 이벤트도 있습니다.

</details>

**Q2. 기존 Chat Completions 코드를 바로 지워야 하나요?**

<details>
<summary>답</summary>

호환 서비스 계약을 확인한 뒤 점진적으로 전환합니다. 새 OpenAI 통합은 Responses API를 우선 검토합니다.

</details>

**Q3. `event.delta`를 무조건 누적하면 되나요?**

<details>
<summary>답</summary>

텍스트 delta 타입인지 먼저 확인하고, 완료·오류 이벤트도 별도로 기록해야 합니다.

</details>

## 한 줄 정리

> 스트림은 텍스트만이 아니라 생성 과정의 이벤트 흐름이므로 type을 기준으로 처리합니다.
