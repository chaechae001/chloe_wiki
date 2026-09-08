# Trainer API 학습 흐름

Trainer는 batch 구성, forward, loss, 역전파, optimizer 갱신, 평가와 checkpoint 저장을 표준 루프로 묶습니다. 자동화된 루프일수록 데이터·모델의 계약을 명확히 해야 합니다.

**핵심 키워드:** Trainer, TrainingArguments, data collator, checkpoint, callback

## 구성 요소

```text
model + TrainingArguments + train/eval Dataset
+ processing_class + data_collator + compute_metrics
→ train() → evaluate() → save_model()
```

```python
from transformers import DataCollatorWithPadding, Trainer, TrainingArguments

args = TrainingArguments(
    output_dir="./artifacts/demo-run",
    num_train_epochs=2,
    per_device_train_batch_size=8,
    per_device_eval_batch_size=16,
    learning_rate=2e-5,
    eval_strategy="epoch",
    save_strategy="epoch",
    report_to="none",
)

trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized["train"],
    eval_dataset=tokenized["validation"],
    processing_class=tokenizer,
    data_collator=DataCollatorWithPadding(tokenizer),
    compute_metrics=compute_metrics,
)
```

인자 이름은 Transformers 버전에 따라 달라질 수 있으므로 설치 버전의 공식 API를 확인합니다. 커스텀 모델은 학습 시 loss를 반환하고 Dataset 열과 `forward()` 인자가 맞아야 합니다.

## 학습 실행

`trainer.train()` 뒤 `evaluate()`로 검증 지표를 확인합니다. 최적 모델 선택을 쓰려면 평가·저장 주기와 `metric_for_best_model`, 지표 방향이 일치해야 합니다. checkpoint에는 모델뿐 아니라 optimizer·scheduler와 진행 상태가 포함될 수 있어 재개 정책을 검증합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| epoch vs step | 전체 학습 데이터 1회 순회 vs optimizer 갱신 단위 |
| batch size vs gradient accumulation | 한 forward의 샘플 수 vs 여러 step gradient 누적 |
| model save vs checkpoint | 추론용 모델 자산 vs 학습 재개 상태까지 포함 가능 |

## 직접 해보기

1. Trainer 필수 구성 요소를 나열하세요.
2. OOM에서 유효 batch를 유지하는 방법을 적으세요.
3. best model 설정 조건을 설명하세요.

<details>
<summary>정답 보기</summary>

1. 모델, 학습 설정, 학습 데이터가 기본이며 평가에는 검증 데이터와 지표 함수가 필요합니다.
2. 장치당 batch를 줄이고 gradient accumulation을 늘려 유효 batch를 맞춥니다.
3. 평가·저장 주기를 호환되게 하고 선택 지표와 증가·감소 방향을 정확히 지정합니다.

</details>

## 연결되는 개념

- 이전: [Evaluate로 평가 파이프라인 만들기](05-evaluate-metrics-pipeline.md)
- 다음: [파인튜닝 운영과 재현성](07-finetuning-operations.md)

## 셀프 체크

- [ ] Trainer 구성 요소를 설명한다.
- [ ] 모델·Dataset 계약을 확인한다.
- [ ] 동적 padding을 연결한다.
- [ ] 평가·저장 주기를 맞춘다.
- [ ] checkpoint 재개를 검증한다.

### 복습 질문 및 답변

**Q1. Trainer가 loss를 자동으로 추측하나요?**

<details>
<summary>답</summary>

표준 모델은 labels로 loss를 반환하지만 커스텀 모델은 호환되는 loss 출력이나 사용자 정의 loss가 필요합니다.

</details>

**Q2. 학습 로그의 loss만 낮으면 충분한가요?**

<details>
<summary>답</summary>

아닙니다. validation 지표, 과적합, 오류 사례와 운영 성능을 함께 봐야 합니다.

</details>

**Q3. checkpoint에서 항상 완전히 동일하게 재개되나요?**

<details>
<summary>답</summary>

코드·데이터·환경과 난수 상태가 달라질 수 있어 실제 재개 테스트와 버전 기록이 필요합니다.

</details>

## 한 줄 정리

> Trainer는 학습 루프를 표준화하지만 모델 출력, 데이터 열과 평가·저장 정책은 명시적으로 맞춰야 합니다.
