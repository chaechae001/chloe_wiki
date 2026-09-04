# AutoModelForCausalLM 직접 추론

직접 추론은 토큰화, tensor 장치 이동, `forward()`와 디코딩을 분리해 Pipeline 내부를 관찰하게 합니다. Causal LM은 각 위치에서 다음 토큰의 점수를 계산합니다.

**핵심 키워드:** AutoModelForCausalLM, input_ids, forward, generate, decode

## 단계별 실행

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "sshleifer/tiny-gpt2"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id)

inputs = tokenizer("Machine learning helps", return_tensors="pt")
with torch.inference_mode():
    outputs = model(**inputs)

print(inputs["input_ids"].shape)
print(outputs.logits.shape)
```

logits shape은 `(batch_size, sequence_length, vocabulary_size)`입니다. `forward()`는 점수를 계산하지만 여러 토큰의 생성 루프는 수행하지 않습니다.

```python
generated = model.generate(**inputs, max_new_tokens=12, do_sample=False)
new_tokens = generated[0, inputs["input_ids"].shape[1]:]
print(tokenizer.decode(new_tokens, skip_special_tokens=True))
```

## 장치와 평가 모드

모델과 입력 tensor는 호환되는 장치에 있어야 합니다. `model.eval()`은 dropout 등의 학습 동작을 끄고, `torch.inference_mode()`는 gradient 기록을 생략해 추론 비용을 줄입니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| `forward()` vs `generate()` | 한 번의 모델 계산 vs 반복적인 토큰 선택 루프 |
| eval mode vs inference mode | 계층 동작 전환 vs autograd 기록 비활성화 |
| input tokens vs new tokens | prompt까지 포함된 입력 vs 새로 생성한 suffix |

## 직접 해보기

1. logits의 세 축을 설명하세요.
2. 생성 결과에서 prompt를 제외하는 slice를 작성하세요.
3. CPU/GPU 장치 불일치 오류를 예방하세요.

<details>
<summary>정답 보기</summary>

1. batch, 위치별 sequence, vocabulary 후보 점수입니다.
2. 입력 토큰 길이부터 생성 tensor 끝까지 선택합니다.
3. 모델 배치를 확인하고 입력 tensor를 동일하거나 지원되는 장치로 이동합니다.

</details>

## 연결되는 개념

- 이전: [Pipeline 텍스트 생성 제어](02-pipeline-text-generation.md)
- 다음: [logits와 hidden state 해석](04-logits-hidden-states.md)

## 셀프 체크

- [ ] 직접 추론 단계를 설명한다.
- [ ] logits shape을 해석한다.
- [ ] forward와 generate를 구분한다.
- [ ] 평가·추론 모드를 적용한다.
- [ ] 새 토큰만 분리해 디코딩한다.

### 복습 질문 및 답변

**Q1. `outputs.logits`를 바로 문자열로 바꿀 수 있나요?**

<details>
<summary>답</summary>

아닙니다. 토큰 선택으로 ID를 얻은 뒤 tokenizer로 디코딩해야 합니다.

</details>

**Q2. `model.eval()`이 gradient 계산도 끄나요?**

<details>
<summary>답</summary>

아닙니다. 계층의 평가 동작을 설정하며 gradient 비활성화는 별도의 inference/no-grad 문맥이 담당합니다.

</details>

**Q3. 왜 생성 결과에 입력 prompt가 포함되나요?**

<details>
<summary>답</summary>

Decoder-only 생성은 기존 입력 뒤에 새 토큰을 이어 붙인 전체 시퀀스를 반환하기 때문입니다.

</details>

## 한 줄 정리

> 직접 추론은 토큰화·forward·토큰 선택·디코딩을 분리해 모델 출력을 정확히 관찰하는 방법입니다.
