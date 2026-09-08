# Evaluate로 평가 파이프라인 만들기

평가는 모델이 낸 예측과 정답을 같은 규칙으로 비교하는 반복 가능한 함수여야 합니다. 하나의 점수보다 오류 비용을 반영한 여러 지표와 실행 조건을 함께 기록합니다.

**핵심 키워드:** Evaluate, accuracy, precision, recall, F1, compute_metrics

## 복수 지표 계산

```python
import evaluate

metrics = evaluate.combine(["accuracy", "precision", "recall", "f1"])
result = metrics.compute(
    predictions=[0, 1, 1, 0],
    references=[0, 1, 0, 0],
)
print(result)
```

불균형 분류에서는 averaging 방식과 positive label을 명시합니다. Accuracy가 높아도 소수 class를 하나도 찾지 못할 수 있으므로 confusion matrix와 class별 지표를 함께 봅니다.

## Trainer 콜백 연결

```python
import numpy as np

accuracy = evaluate.load("accuracy")

def compute_metrics(eval_prediction):
    logits, labels = eval_prediction
    predictions = np.argmax(logits, axis=-1)
    return accuracy.compute(predictions=predictions, references=labels)
```

모델 출력 형태가 tuple이거나 token-level이면 전처리가 달라집니다. padding label처럼 평가에서 제외할 위치도 과업 규칙에 맞게 mask 처리합니다.

## 생성 모델 평가

텍스트 겹침 기반 지표는 의미·사실성·안전을 완전히 측정하지 못합니다. 자동 지표, 과업별 규칙, 사람 평가와 오류 사례 분석을 조합합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| metric vs loss | 보고용 성능 기준 vs 학습을 이끄는 미분 가능한 목표 |
| offline vs online 평가 | 고정 데이터셋 성능 vs 실제 트래픽 품질·지연 |
| micro vs macro 평균 | 전체 사례 기여 중심 vs class별 동일 비중 |

## 직접 해보기

1. 사기 탐지에 중요한 지표를 고르세요.
2. logits에서 class 예측을 만드는 축을 설명하세요.
3. 생성 평가표에 넣을 항목을 설계하세요.

<details>
<summary>정답 보기</summary>

1. 놓친 사기의 비용이 크면 Recall을 중시하되 Precision과 임계값 비용도 함께 봅니다.
2. 각 샘플의 label 차원에서 가장 큰 logit의 인덱스를 선택합니다.
3. 자동 지표, 사실 오류, 유해 출력, 사람 선호, 지연과 비용을 함께 기록합니다.

</details>

## 연결되는 개념

- 이전: [커스텀 데이터셋과 토큰화 전처리](04-custom-dataset-preprocessing.md)
- 다음: [Trainer API 학습 흐름](06-trainer-api-workflow.md)

## 셀프 체크

- [ ] 과업 비용에 맞는 지표를 고른다.
- [ ] 여러 지표를 같은 예측에서 계산한다.
- [ ] logits 후처리를 검증한다.
- [ ] 불균형과 평균 방식을 확인한다.
- [ ] 자동 지표의 한계를 기록한다.

### 복습 질문 및 답변

**Q1. F1이 높으면 모든 class 성능이 좋은가요?**

<details>
<summary>답</summary>

평균 방식에 따라 소수 class 성능이 가려질 수 있어 class별 값도 확인해야 합니다.

</details>

**Q2. test 결과를 보고 학습률을 바꿔도 되나요?**

<details>
<summary>답</summary>

그러면 test가 튜닝에 사용되어 최종 평가가 낙관적으로 편향됩니다. validation을 사용합니다.

</details>

**Q3. 지표 계산도 버전 관리해야 하나요?**

<details>
<summary>답</summary>

예. 데이터 revision, 전처리, metric 설정과 라이브러리 버전이 결과에 영향을 줍니다.

</details>

## 한 줄 정리

> 좋은 평가 파이프라인은 예측 변환과 지표 조건을 고정하고 실제 오류 비용을 여러 관점에서 보여줍니다.
