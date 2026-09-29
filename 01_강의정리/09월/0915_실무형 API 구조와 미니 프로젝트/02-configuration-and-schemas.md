# 설정과 Schema 설계

> 실행 환경의 값과 API 계약을 코드에서 분리하면 누락을 일찍 발견하고 입력 경계를 명확히 만들 수 있습니다.

`environment variable` · `Pydantic` · `fail fast` · `validation` · `secret`

## 핵심요약

- API key와 model 설정은 환경 변수 또는 secret manager에서 읽습니다.
- 애플리케이션 시작 시 필수 설정을 검증해 빠르게 실패시킵니다.
- `.env.example`에는 변수 이름만 공유하고 실제 비밀은 넣지 않습니다.
- Pydantic은 타입, 길이, 범위를 HTTP 경계에서 검증합니다.
- 사용자가 model과 생성 옵션을 자유롭게 선택하지 않도록 허용 정책을 둡니다.

## 1. 설정을 한곳에 모으기

```python
import os
from openai import AsyncOpenAI

API_KEY = os.environ["OPENAI_API_KEY"]
MODEL = os.environ["OPENAI_MODEL"]

client = AsyncOpenAI(api_key=API_KEY)
```

비밀값은 브라우저, notebook output, Git history, 오류 응답에 남기지 않습니다. 노출 가능성이 생기면 단순 삭제가 아니라 키 폐기와 재발급이 필요합니다.

<details>
<summary>답</summary>

시작 단계에서 누락을 확인하면 첫 사용자 요청에서 늦게 500 오류가 발생하는 대신 배포 직후 문제를 발견할 수 있습니다.

</details>

## 2. 요청 계약

```python
from pydantic import BaseModel, Field

class ChatRequest(BaseModel):
    message: str = Field(min_length=1, max_length=4000)
    temperature: float = Field(default=0.3, ge=0.0, le=2.0)
```

공백 문자열은 길이가 1 이상일 수 있으므로 `strip()` validator 또는 Service의 의미 검증을 추가합니다. Model 선택권을 제공한다면 server allowlist로 제한합니다.

## 3. 응답 계약

```python
class ChatResponse(BaseModel):
    reply: str
    request_id: str
    cached: bool = False
```

Provider 응답 객체를 그대로 외부에 반환하지 않고 서비스가 약속할 field만 정의합니다. 이렇게 하면 SDK 교체가 공개 API breaking change로 이어지지 않습니다.

## 직접 해보기

1. 실제 값이 없는 `.env.example`을 작성하세요.
2. 공백만 있는 message를 거절하세요.
3. 응답에서 provider 내부 필드를 제거하세요.

<details>
<summary>정답 보기</summary>

변수 이름만 공유하고, `if not message.strip()`으로 의미 검증합니다. 응답은 `reply`, `request_id`, `cached`처럼 서비스 계약으로 다시 조립합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `.env` vs secret manager | 로컬 파일 기반 설정과 배포 환경의 비밀 저장소입니다. |
| 422 vs 400 | Schema 불일치와 의미상 잘못된 입력입니다. |
| Provider model vs public model option | 내부 선택과 사용자가 요청할 수 있는 허용 값입니다. |

## 연결되는 개념

- 이전: [실무형 레이어드 API 구조](01-layered-api-architecture.md)
- 다음: [LLM Client와 Service 모듈화](03-llm-client-service-modularization.md)

## 셀프 체크

- [ ] 비밀값을 환경 설정으로 분리한다.
- [ ] 필수 설정 누락을 시작 시 감지한다.
- [ ] 입력 길이와 범위를 검증한다.
- [ ] 공백 입력을 별도로 처리한다.
- [ ] 공개 응답과 provider 객체를 분리한다.

### 복습 질문 및 답변

**Q1. `.gitignore`를 추가하면 이미 커밋한 키도 사라지나요?**

<details>
<summary>답</summary>

아닙니다. Git history에서 별도로 처리하고 키를 즉시 교체해야 합니다.

</details>

**Q2. 모든 환경 변수를 필수로 만들어야 하나요?**

<details>
<summary>답</summary>

보안·기능상 필수 값은 강제하고 안전한 기본값이 있는 일반 설정만 default를 둘 수 있습니다.

</details>

**Q3. Response model이 왜 필요한가요?**

<details>
<summary>답</summary>

응답 field를 검증하고 문서화하며 내부 정보가 우연히 노출되는 것을 막습니다.

</details>

## 한 줄 정리

> 설정은 실행 환경의 계약이고 Schema는 외부 요청·응답의 계약입니다.
