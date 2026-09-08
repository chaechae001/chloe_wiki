# 사전학습 백본에 커스텀 헤드 추가하기

사전학습 Transformer는 문맥 표현을 만드는 백본입니다. 커스텀 헤드는 이 표현을 분류·회귀처럼 원하는 출력으로 바꾸는 작은 과업 계층입니다.

**핵심 키워드:** backbone, task head, PreTrainedModel, logits, loss

## 기본 구조

```text
input_ids·attention_mask → pretrained backbone → hidden states
→ pooling 또는 특정 토큰 선택 → dropout → linear head → logits
```

```python
import torch.nn as nn
from transformers import AutoModel

class TextClassifier(nn.Module):
    def __init__(self, model_id, num_labels):
        super().__init__()
        self.backbone = AutoModel.from_pretrained(model_id)
        self.dropout = nn.Dropout(0.1)
        self.head = nn.Linear(self.backbone.config.hidden_size, num_labels)

    def forward(self, input_ids, attention_mask):
        output = self.backbone(input_ids=input_ids, attention_mask=attention_mask)
        pooled = output.last_hidden_state[:, 0]
        return self.head(self.dropout(pooled))
```

첫 토큰 표현이 항상 최선의 문장 표현인 것은 아닙니다. 모델의 특수 토큰·pooler 구조와 과업을 확인해 mean pooling 같은 대안을 비교합니다.

## Transformers 규약과 연결

`PreTrainedModel`을 상속하면 config, `save_pretrained()`와 가중치 초기화 규약을 활용하기 쉽습니다. Trainer에 연결하려면 `forward()`가 입력 컬럼을 받고, 학습 시 `loss`와 `logits`를 포함한 호환 가능한 출력을 반환하도록 설계합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 백본 vs 헤드 | 일반 표현 추출 vs 과업별 예측 |
| logits vs label | 정규화 전 점수 vs 최종 정답 범주 |
| 분류 vs 회귀 | 범주별 점수와 CrossEntropy vs 연속값과 회귀 손실 |

## 직접 해보기

1. hidden size 768, 라벨 4개의 Linear 파라미터 수를 계산하세요.
2. mean pooling에서 padding을 제외하는 방법을 설명하세요.
3. 백본 동결 실험을 설계하세요.

<details>
<summary>정답 보기</summary>

1. weight 3,072개와 bias 4개로 총 3,076개입니다.
2. attention mask를 가중치로 곱해 실제 토큰만 합산하고 실제 토큰 수로 나눕니다.
3. 백본의 `requires_grad`를 끄고 헤드만 학습한 기준선과 전체 미세조정을 같은 검증 세트로 비교합니다.

</details>

## 연결되는 개념

- 다음: [멀티태스크 학습과 앙상블](02-multitask-and-ensembles.md)
- 함께 볼 키워드: `pooling`, `ModelOutput`, `freeze`

## 셀프 체크

- [ ] 백본과 헤드의 책임을 구분한다.
- [ ] 출력 shape를 계산한다.
- [ ] 모델에 맞는 pooling을 선택한다.
- [ ] loss 반환 규약을 설명한다.
- [ ] 동결 여부를 실험으로 결정한다.

### 복습 질문 및 답변

**Q1. 커스텀 헤드는 반드시 커야 하나요?**

<details>
<summary>답</summary>

아닙니다. 단순 Linear head도 강한 기준선이며 복잡도는 검증 성능으로 결정합니다.

</details>

**Q2. 라벨 수가 바뀌면 무엇이 달라지나요?**

<details>
<summary>답</summary>

마지막 head의 출력 차원과 config, 손실 계산 및 라벨 매핑을 함께 맞춰야 합니다.

</details>

**Q3. 백본 가중치를 전부 갱신하면 항상 좋은가요?**

<details>
<summary>답</summary>

작은 데이터에서는 과적합과 비용이 커질 수 있어 동결·부분 미세조정과 비교해야 합니다.

</details>

## 한 줄 정리

> 커스텀 헤드는 사전학습 표현을 과업 출력으로 바꾸며 입력·출력·loss 규약이 학습 도구와의 연결점입니다.
