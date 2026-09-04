# logits와 hidden state 해석

모델 출력은 완성 문장이 아니라 위치별 logits와 선택적인 hidden state·attention·cache입니다. shape와 변환 과정을 알면 다음 토큰 선택을 검증할 수 있습니다.

**핵심 키워드:** logits, softmax, top-k, hidden state, vocabulary

## 다음 토큰 확률

```python
import torch

next_logits = outputs.logits[:, -1, :]
probabilities = torch.softmax(next_logits, dim=-1)
top_prob, top_id = torch.topk(probabilities, k=5, dim=-1)

for token_id, probability in zip(top_id[0], top_prob[0]):
    token = tokenizer.decode([token_id.item()])
    print(repr(token), round(probability.item(), 4))
```

마지막 입력 위치의 vocabulary 점수만 골라 softmax로 합이 1인 분포로 바꿉니다. top-k는 전체 분포 중 높은 후보만 관찰하는 분석 도구이며 그 자체가 생성 전략의 `top_k`와 항상 같은 사용 맥락은 아닙니다.

## hidden state

기본 모델의 `last_hidden_state`는 보통 `(batch, sequence, hidden_size)` 형태의 문맥 표현입니다. Causal LM의 logits는 hidden state에 language-model head를 적용한 결과입니다. 모든 출력이 기본 반환되는 것은 아니며 hidden states와 attentions는 옵션을 켜야 해 메모리 사용이 늘 수 있습니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| logits vs probability | 정규화 전 실수 점수 vs softmax 후 분포 |
| hidden size vs vocabulary size | 내부 표현 차원 vs 후보 토큰 수 |
| top-k 관찰 vs top-k sampling | 상위 후보 출력 분석 vs 생성 후보 제한 |

## 직접 해보기

1. 다음 토큰 위치를 선택하는 코드를 설명하세요.
2. softmax 축을 잘못 고르면 생길 문제를 적으세요.
3. hidden state 활용 사례를 제시하세요.

<details>
<summary>정답 보기</summary>

1. sequence의 마지막 위치 `-1`에서 모든 vocabulary 점수를 선택합니다.
2. 토큰 후보가 아닌 위치나 batch 사이를 정규화해 의미 없는 확률이 됩니다.
3. 분류 head 입력, 토큰 표현 분석이나 유사도 특징에 활용할 수 있습니다.

</details>

## 연결되는 개념

- 이전: [AutoModelForCausalLM 직접 추론](03-causal-lm-direct-inference.md)
- 다음: [생성 전략과 스트리밍](05-generation-and-streaming.md)

## 셀프 체크

- [ ] logits의 shape을 설명한다.
- [ ] 올바른 축으로 softmax를 적용한다.
- [ ] 상위 토큰을 문자열로 확인한다.
- [ ] hidden과 vocabulary 차원을 구분한다.
- [ ] 선택 출력의 메모리 비용을 안다.

### 복습 질문 및 답변

**Q1. 가장 큰 logit의 토큰이 항상 생성되나요?**

<details>
<summary>답</summary>

Greedy에서는 그렇지만 sampling이나 제약 조건을 쓰면 다른 토큰이 선택될 수 있습니다.

</details>

**Q2. softmax를 해도 토큰 순위가 바뀌나요?**

<details>
<summary>답</summary>

같은 축의 유한 logits에 적용하면 단조 변환이므로 일반적으로 순위는 유지됩니다.

</details>

**Q3. attention weight가 모델의 완전한 설명인가요?**

<details>
<summary>답</summary>

아닙니다. 일부 내부 신호이며 출력 원인을 완전히 설명한다고 단정할 수 없습니다.

</details>

## 한 줄 정리

> logits는 다음 토큰 후보의 원점수이고 hidden state는 그 점수를 만들기 전의 문맥 표현입니다.
