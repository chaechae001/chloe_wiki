# Pipeline 텍스트 생성 제어

텍스트 생성 Pipeline은 prompt나 채팅 메시지를 받아 autoregressive 모델의 `generate()`를 호출합니다. 생성 길이와 토큰 선택 전략을 명시해야 결과를 비교할 수 있습니다.

**핵심 키워드:** text-generation, max_new_tokens, sampling, temperature, chat

## 작은 생성 예제

```python
from transformers import pipeline

generator = pipeline("text-generation", model="sshleifer/tiny-gpt2")
result = generator(
    "A useful model evaluation starts with",
    max_new_tokens=20,
    do_sample=False,
    return_full_text=False,
)
print(result[0]["generated_text"])
```

작은 모델은 실행 흐름 확인에 적합하지만 품질 기준 모델로 해석하지 않습니다. `return_full_text=False`는 새로 생성된 부분만 보기 쉽게 합니다.

## 생성 옵션

| 옵션 | 역할 | 주의점 |
|---|---|---|
| `max_new_tokens` | 새로 만들 토큰 수 상한 | 입력 길이를 포함하지 않음 |
| `do_sample` | 확률적 샘플링 사용 | 같은 입력도 결과가 달라질 수 있음 |
| `temperature` | 분포의 평탄함 조절 | sampling을 쓸 때 의미가 큼 |
| `top_k`, `top_p` | 후보 토큰 범위 제한 | 품질·다양성에 함께 영향 |

채팅 모델에는 문자열보다 `role`과 `content` 메시지 배열을 전달할 수 있습니다. 해당 tokenizer의 chat template 지원 여부를 확인합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| `max_length` vs `max_new_tokens` | 입력을 포함한 전체 길이 vs 생성 부분 길이 |
| greedy vs sampling | 매번 최고 점수 선택 vs 분포에서 확률적 선택 |
| prompt string vs chat | 단일 텍스트 입력 vs 역할이 있는 대화 구조 |

## 직접 해보기

1. 결정적 결과가 필요한 설정을 적으세요.
2. 입력이 긴 경우 생성 길이 옵션을 선택하세요.
3. 세 생성 설정의 비교표를 설계하세요.

<details>
<summary>정답 보기</summary>

1. `do_sample=False`를 사용하고 버전과 입력을 고정합니다.
2. 의도가 명확한 `max_new_tokens`를 우선 검토하고 전체 context 한계도 확인합니다.
3. 설정, 생성 텍스트, 토큰 수, 지연과 반복 여부를 기록합니다.

</details>

## 연결되는 개념

- 이전: [Pipeline API로 빠르게 추론하기](01-pipeline-api-basics.md)
- 다음: [AutoModelForCausalLM 직접 추론](03-causal-lm-direct-inference.md)

## 셀프 체크

- [ ] 생성 Pipeline의 입출력을 설명한다.
- [ ] 두 길이 옵션을 구분한다.
- [ ] greedy와 sampling을 구분한다.
- [ ] 채팅 입력 형식을 확인한다.
- [ ] 설정과 지연을 함께 기록한다.

### 복습 질문 및 답변

**Q1. `temperature=0`만 쓰면 항상 올바른 결정적 설정인가요?**

<details>
<summary>답</summary>

라이브러리와 전략에 따라 다르므로 결정적 생성은 `do_sample=False`를 명시하는 편이 분명합니다.

</details>

**Q2. 토큰 수와 글자 수는 같은가요?**

<details>
<summary>답</summary>

아닙니다. 토크나이저 분절에 따라 한 글자·단어가 여러 토큰이 될 수 있습니다.

</details>

**Q3. 작은 모델의 어색한 결과는 코드 오류인가요?**

<details>
<summary>답</summary>

반드시 그렇지는 않습니다. 모델 용량·학습 목적과 prompt 적합성의 한계일 수 있어 흐름과 품질을 분리해 봅니다.

</details>

## 한 줄 정리

> 텍스트 생성은 모델뿐 아니라 길이와 디코딩 설정을 기록해야 비교 가능한 실험이 됩니다.
