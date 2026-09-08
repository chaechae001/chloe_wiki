# PEFT와 LoRA 어댑터

대형 모델 전체를 다시 학습하지 않고도 작은 변화량만 학습하면 비용을 크게 줄일 수 있습니다. PEFT는 학습 대상을 제한하고, LoRA는 가중치 변화량을 저랭크 행렬로 표현합니다.

**핵심 키워드:** PEFT, LoRA, low rank, adapter, trainable parameters

## LoRA의 핵심 원리

기존 가중치 $W$는 고정하고 학습할 변화량을 두 작은 행렬의 곱으로 나타냅니다.

$$
y = Wx + \frac{\alpha}{r}BAx
$$

$A \in \mathbb{R}^{r \times d_{in}}$, $B \in \mathbb{R}^{d_{out} \times r}$이며 $r$은 작은 rank입니다. 전체 변화량 행렬 대신 $A$와 $B$만 학습하므로 trainable parameter가 줄어듭니다.

```python
from peft import LoraConfig, TaskType, get_peft_model

config = LoraConfig(
    task_type=TaskType.SEQ_CLS,
    r=8,
    lora_alpha=16,
    lora_dropout=0.05,
    target_modules=["query", "value"],
)
model = get_peft_model(base_model, config)
model.print_trainable_parameters()
```

모듈 이름은 모델 구조마다 다릅니다. `target_modules`를 추측하지 말고 `named_modules()`로 실제 이름을 확인합니다.

## 초기화와 학습 대상 검증

일반적인 LoRA 초기화는 한 행렬을 0으로 두어 시작 시 adapter 변화량이 0이 되도록 합니다. 이때 첫 출력은 base 모델과 같고 학습이 진행되며 변화량이 생깁니다.

검증할 항목은 다음과 같습니다.

- base 파라미터가 동결되었는가
- LoRA 파라미터만 optimizer에 들어갔는가
- trainable 비율이 의도한 수준인가
- adapter 저장 후 같은 base 모델에서 재로드되는가

```python
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print({"trainable": trainable, "ratio": trainable / total})
```

## Rank와 적용 위치

rank가 커지면 표현력과 비용이 함께 늘어납니다. 모든 Linear에 무조건 붙이기보다 attention projection이나 과업에 중요한 모듈부터 비교합니다. adapter만 작아도 forward에는 base 모델 전체가 필요하므로 추론 모델 자체가 자동으로 작아지는 것은 아닙니다.

## 직접 해보기

1. 입력 768, 출력 768인 층에 rank 8 LoRA를 붙일 때 bias를 제외한 추가 파라미터 수를 구하세요.
2. adapter 학습 후 base 가중치가 유지됐는지 검사하는 방법을 설명하세요.
3. 여러 고객별 adapter를 운영하는 저장 전략을 설계하세요.

<details>
<summary>정답 보기</summary>

1. $768\times8 + 8\times768 = 12{,}288$개입니다.
2. 학습 전후 base state dict를 비교하고 base 파라미터의 `requires_grad=False` 및 gradient 부재를 확인합니다.
3. 공통 base 버전을 고정하고 고객별 adapter·설정·평가 결과만 별도 버전으로 관리합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| PEFT vs LoRA | 효율적 미세조정의 범주 vs 그 안의 저랭크 기법 |
| rank vs alpha | adapter 용량 vs update 크기 조절 |
| adapter 저장 vs 병합 저장 | 작은 변화량만 저장 vs base에 합친 전체 가중치 저장 |

## 연결되는 개념

- 이전: [체크포인트 복구와 메모리 최적화](02-checkpoints-and-memory.md)
- 다음: [양자화 전략과 실행 환경](04-quantization-strategies.md)
- 함께 볼 키워드: `QLoRA`, `target_modules`, `merge`

## 셀프 체크

- [ ] LoRA 수식을 shape와 함께 설명한다.
- [ ] trainable parameter 수를 계산한다.
- [ ] target module을 실제 구조에서 찾는다.
- [ ] base 동결 여부를 검증한다.
- [ ] adapter와 전체 모델 저장을 구분한다.

### 복습 질문 및 답변

**Q1. LoRA는 base 모델을 삭제하나요?**

<details>
<summary>답</summary>

아닙니다. base 출력에 학습된 저랭크 변화량을 더하므로 adapter 방식 추론에는 호환되는 base 모델이 필요합니다.

</details>

**Q2. rank가 클수록 항상 좋은가요?**

<details>
<summary>답</summary>

표현력은 늘지만 메모리·저장 비용과 과적합 가능성도 커지므로 검증 성능으로 선택해야 합니다.

</details>

**Q3. loss가 내려가면 PEFT 구성이 맞다는 뜻인가요?**

<details>
<summary>답</summary>

아닙니다. base까지 실수로 학습해도 loss는 내려갈 수 있어 학습 파라미터 목록과 저장 결과를 별도로 검사해야 합니다.

</details>

## 한 줄 정리

> LoRA는 고정된 base 모델에 작은 저랭크 변화량을 학습해 비용과 재사용성을 함께 관리하는 PEFT 기법입니다.
