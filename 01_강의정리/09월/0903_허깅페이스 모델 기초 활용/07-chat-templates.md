# 채팅 템플릿과 대화 입력

채팅 모델도 결국 다음 토큰을 예측합니다. 채팅 템플릿은 `role`과 `content` 목록을 모델이 학습한 제어 토큰·구분자 형식의 단일 토큰 시퀀스로 바꿉니다.

**핵심 키워드:** chat template, Jinja, role, apply_chat_template, generation prompt

## 메시지 구조

```python
messages = [
    {"role": "system", "content": "Answer briefly and clearly."},
    {"role": "user", "content": "Why is padding needed?"},
]

inputs = tokenizer.apply_chat_template(
    messages,
    tokenize=True,
    add_generation_prompt=True,
    return_tensors="pt",
)
print(inputs.shape)
```

`tokenize=True`는 바로 모델에 전달할 ID를 만듭니다. `add_generation_prompt=True`는 모델 템플릿이 지원한다면 다음 응답이 assistant 차례임을 알리는 시작 표식을 덧붙입니다.

## 모델별 형식

같은 대화라도 모델마다 역할 토큰, 줄바꿈과 종료 토큰이 다릅니다. 다른 모델의 문자열 포맷을 수동 복사하지 말고 해당 tokenizer의 `chat_template`을 사용합니다. 모든 모델이 system role이나 generation prompt를 같은 방식으로 지원하는 것은 아닙니다.

## 중복 특수 토큰 주의

`apply_chat_template(tokenize=False)`로 문자열을 만든 뒤 다시 토큰화한다면 템플릿이 이미 넣은 BOS/EOS를 중복 추가하지 않도록 `add_special_tokens=False`를 검토합니다. 가능하면 템플릿에서 곧바로 토큰화하는 흐름이 단순합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| messages vs prompt string | 구조화된 역할 목록 vs 렌더링된 최종 문자열 |
| generation prompt vs 답변 내용 | assistant 시작 표식 vs 실제 생성된 토큰 |
| `tokenize=True` vs `False` | 토큰 ID 반환 vs 렌더링 문자열 반환 |

## 직접 해보기

1. system-user-assistant-user 멀티턴 메시지를 작성하세요.
2. generation prompt 유무의 마지막 토큰을 비교하세요.
3. 두 모델의 렌더링 문자열이 다른 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 각 발화를 순서대로 `role`, `content` 사전에 넣습니다.
2. 지원 모델에서는 활성화 시 assistant 응답 시작 표식이 추가됩니다.
3. 모델이 학습될 때 사용한 역할 토큰과 대화 구분 형식이 다르기 때문입니다.

</details>

## 연결되는 개념

- 이전: [배치, 패딩과 트런케이션](06-padding-truncation-attention-mask.md)
- 처음으로: [모델 저장소 파일과 설정 읽기](01-model-repository-files.md)

## 셀프 체크

- [ ] messages 표준 구조를 작성한다.
- [ ] 템플릿이 필요한 이유를 설명한다.
- [ ] 반환 문자열과 ID를 구분한다.
- [ ] generation prompt의 역할을 안다.
- [ ] 특수 토큰 중복을 방지한다.

### 복습 질문 및 답변

**Q1. 모든 채팅 모델에 같은 템플릿을 써도 되나요?**

<details>
<summary>답</summary>

아닙니다. 학습 형식이 달라 해당 모델 토크나이저의 템플릿을 따라야 합니다.

</details>

**Q2. `add_generation_prompt=True`가 답변을 생성하나요?**

<details>
<summary>답</summary>

아닙니다. 다음이 assistant 응답임을 표시할 뿐 실제 생성은 모델의 `generate`가 수행합니다.

</details>

**Q3. 템플릿 결과를 디코딩해 보는 이유는 무엇인가요?**

<details>
<summary>답</summary>

역할 순서, 제어 토큰, 종료 표식과 중복 특수 토큰이 의도대로인지 확인할 수 있습니다.

</details>

## 한 줄 정리

> 채팅 템플릿은 구조화된 대화를 모델이 학습한 정확한 토큰 문법으로 번역합니다.
