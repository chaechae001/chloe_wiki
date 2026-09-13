# Python SDK와 Responses API

> SDK는 인증·직렬화·응답 객체 처리를 단순화하고 Responses API는 새 통합의 기본 호출 경로를 제공합니다.

`OpenAI client` · `Responses API` · `input` · `instructions` · `output_text`

## 핵심요약

- 클라이언트는 애플리케이션 생명주기 동안 재사용합니다.
- 현재 OpenAI 직접 호출은 Responses API를 기본 선택으로 봅니다.
- 행동 규칙은 `instructions`, 사용자 요청은 `input`으로 분리할 수 있습니다.
- SDK의 `output_text` helper로 여러 output item의 텍스트를 안전하게 모읍니다.
- 모델 식별자는 코드에 고정하지 않고 환경별 설정으로 둡니다.

## 1. 가장 작은 호출

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

response = client.responses.create(
    model=os.environ["OPENAI_MODEL"],
    instructions="핵심만 세 문장으로 설명하세요.",
    input="API endpoint란 무엇인가요?",
)

print(response.output_text)
```

SDK가 HTTP header와 JSON 변환을 처리하지만, timeout·재시도·오류 정책까지 자동으로 서비스 요구사항에 맞춰 주는 것은 아닙니다.

<details>
<summary>답</summary>

`output`은 텍스트 이외의 item도 담을 수 있으므로 텍스트만 필요하면 SDK의 `output_text` helper가 직접 배열 위치를 가정하는 것보다 안전합니다.

</details>

## 2. 지침과 입력 분리

`instructions`는 답변 방식과 역할을, `input`은 이번 요청을 나타냅니다. 사용자가 서버 정책을 덮어쓰지 못하도록 시스템 지침을 요청 body에서 그대로 받지 않는 설계가 보통 더 안전합니다.

## 3. Chat Completions 호환 호출

기존 코드나 호환 게이트웨이는 `client.chat.completions.create(messages=[...])`를 사용할 수 있습니다. 새 OpenAI 통합과 호환 서비스 요구사항을 같은 것으로 가정하지 말고, 제공자 문서와 endpoint contract를 확인합니다.

## 코드로 보기 — 호출 함수 경계

### 코드 목적

웹 endpoint에서 SDK 세부사항을 분리합니다.

### 코드 흐름

1. 함수가 사용자 텍스트를 받습니다.
2. SDK를 호출합니다.
3. 필요한 텍스트만 반환합니다.

```python
def generate_reply(message: str) -> str:
    result = client.responses.create(
        model=os.environ["OPENAI_MODEL"],
        instructions="친절한 기술 조교로 답하세요.",
        input=message,
    )
    return result.output_text
```

### 실행 결과 해석

함수 호출자는 공급자 응답의 전체 구조 대신 문자열 계약에만 의존합니다.

### 실무 연결

이 경계에 tracing, timeout, 비용 기록과 provider 교체 로직을 붙일 수 있습니다.

## 직접 해보기

1. 지침과 입력을 분리한 호출을 작성하세요.
2. 모델 이름을 환경 변수로 옮기세요.
3. SDK 객체 전체 대신 텍스트만 반환하는 함수를 만드세요.

<details>
<summary>정답 보기</summary>

`client.responses.create(model=os.environ["OPENAI_MODEL"], instructions=..., input=...)`로 호출하고 `response.output_text`를 반환하면 됩니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Responses vs Chat Completions | 새 OpenAI 통합의 기본 API와 기존 대화 메시지 형식의 API입니다. |
| `instructions` vs `input` | 응답 규칙과 이번 사용자 요청입니다. |
| SDK vs REST 직접 호출 | 편의 추상화와 HTTP 요청을 직접 조립하는 방식입니다. |

## 연결되는 개념

- 이전: [API 키와 서버 보안](01-api-key-and-server-security.md)
- 다음: [입력·출력과 대화 상태](03-input-output-and-conversation-state.md)

## 셀프 체크

- [ ] OpenAI client 생성 위치를 설명한다.
- [ ] `instructions`와 `input`을 구분한다.
- [ ] `output_text`의 목적을 안다.
- [ ] 모델 이름을 설정으로 분리한다.
- [ ] 호환 API와 공식 endpoint 차이를 확인한다.

### 복습 질문 및 답변

**Q1. 요청마다 client를 새로 만들어야 하나요?**

<details>
<summary>답</summary>

일반적으로 설정된 client를 재사용해 연결과 구성을 한곳에서 관리합니다.

</details>

**Q2. `response.output[0]`만 읽으면 충분한가요?**

<details>
<summary>답</summary>

출력 item 구성이 달라질 수 있으므로 텍스트 목적에는 `output_text` helper가 더 견고합니다.

</details>

**Q3. SDK를 쓰면 보안 처리가 끝나나요?**

<details>
<summary>답</summary>

아닙니다. 키 저장, 입력 검증, 권한, logging과 오류 노출 정책은 애플리케이션 책임입니다.

</details>

## 한 줄 정리

> SDK 호출을 작은 함수로 감싸고 Responses API의 입력과 출력 계약을 명확히 다룹니다.
