# System Prompt와 Template

> 프롬프트를 호출 코드에서 분리하면 행동 정책을 찾고 비교하고 버전 관리하기 쉬워집니다.

`instructions` · `developer message` · `template` · `prompt version` · `evaluation`

## 핵심요약

- System/developer instruction은 제품의 역할·톤·언어·제약을 정의합니다.
- 사용자 입력과 서버 instruction을 별도 field로 유지합니다.
- 템플릿 변수는 명시적으로 검증하고 사용자 입력을 정책 문자열에 직접 결합하지 않습니다.
- 프롬프트 변경은 코드 변경처럼 version과 평가 기록을 남깁니다.
- 재사용되는 안정적인 prefix는 prompt caching에도 유리합니다.

## 1. Prompt 분리

```python
SYSTEM_INSTRUCTION = (
    "당신은 친절한 기술 튜터입니다. "
    "한국어로 핵심을 세 문장 이내로 설명하세요."
)
```

Responses API에서는 `instructions`에 행동 규칙을, `input`에 사용자 요청을 전달할 수 있습니다. 이전 response를 이어도 과거 instructions가 자동으로 이어진다고 가정하지 않고 요청 정책을 명시합니다.

<details>
<summary>답</summary>

서버 정책과 사용자 입력을 한 문자열에 섞으면 우선순위와 변경 추적이 흐려지고 prompt injection 대응도 어려워집니다.

</details>

## 2. Template 설계

```python
TEMPLATE = "주제: {topic}\n독자 수준: {level}\n설명해 주세요."

def build_prompt(topic: str, level: str) -> str:
    if level not in {"beginner", "intermediate"}:
        raise ValueError("unsupported level")
    return TEMPLATE.format(topic=topic, level=level)
```

문자열 보간 자체가 보안 경계는 아닙니다. 허용값, 길이, 출력 schema를 별도로 검증합니다.

## 3. Version과 평가

프롬프트 ID·version 또는 Git commit을 request log와 연결하면 어떤 문구가 결과를 만들었는지 재현할 수 있습니다. 변경 전후에 대표 질문 dataset으로 정확성, 형식 준수, 거절 동작을 비교합니다.

## 직접 해보기

1. 역할·톤·언어·제약 순서로 instruction을 작성하세요.
2. 허용된 독자 수준만 받는 template 함수를 만드세요.
3. 프롬프트 version을 log field에 추가하세요.

<details>
<summary>정답 보기</summary>

안정적인 instruction과 가변 사용자 입력을 분리하고, template 변수는 allowlist로 검증합니다. `prompt_version`을 구조화 log에 기록하면 회귀 원인을 찾기 쉽습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Instruction vs user input | 제품 행동 규칙과 매 요청의 작업입니다. |
| Template vs 완성 prompt | 변수 자리가 있는 형식과 값이 채워진 입력입니다. |
| 변경 vs 개선 | 문구 수정 사실과 평가로 확인한 성능 향상입니다. |

## 연결되는 개념

- 이전: [LLM Client와 Service 모듈화](03-llm-client-service-modularization.md)
- 다음: [Logging과 관측성](05-logging-and-observability.md)

## 셀프 체크

- [ ] Instruction과 user input을 분리한다.
- [ ] Template 변수를 검증한다.
- [ ] Prompt version을 추적한다.
- [ ] 변경 전후 평가를 수행한다.
- [ ] 안정적인 prefix를 앞쪽에 둔다.

### 복습 질문 및 답변

**Q1. System Prompt가 길수록 좋은가요?**

<details>
<summary>답</summary>

아닙니다. 필요한 규칙을 명확하고 충돌 없이 작성하고 평가로 효과를 확인해야 합니다.

</details>

**Q2. 사용자 입력을 그대로 template에 넣어도 되나요?**

<details>
<summary>답</summary>

입력 길이와 형식을 검증하고 instruction과 분리해야 합니다. Template 보간만으로 injection을 막을 수는 없습니다.

</details>

**Q3. Prompt caching과 앱의 응답 캐시는 같은가요?**

<details>
<summary>답</summary>

아닙니다. Prompt caching은 동일 prefix 처리 작업을 재사용하고, 응답 캐시는 완성된 결과를 직접 반환합니다.

</details>

## 한 줄 정리

> 프롬프트는 코드 밖의 문장이 아니라 version·검증·평가가 필요한 제품 정책입니다.
