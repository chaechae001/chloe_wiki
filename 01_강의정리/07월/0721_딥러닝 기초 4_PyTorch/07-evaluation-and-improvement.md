# 모델 평가와 성능 개선

> 좋은 모델은 학습 손실만 낮은 모델이 아니라 보지 않은 데이터에서도 안정적으로 예측하는 모델입니다.

`validation` · `test` · `accuracy` · `overfitting` · `hyperparameter`

## 핵심요약

- 학습과 검증 지표를 함께 기록해 일반화 흐름을 관찰한다.
- 테스트 데이터는 최종 모델 선택 이후 한 번의 독립 평가에 가깝게 사용한다.
- 손실과 정확도는 서로 다른 정보를 제공한다.
- 학습률, 배치 크기, 에포크 수는 검증 성능으로 조정한다.
- 정규화, 데이터 증강, Dropout은 문제에 맞게 실험하고 결과를 비교한다.

## 1. 검증 지표 기록

```python
history = {
    "train_loss": [],
    "val_loss": [],
    "val_acc": [],
}
```

학습 손실은 계속 낮아지는데 검증 손실이 다시 높아지면 과적합 가능성을 의심합니다.

## 2. 정확한 평가 함수

```python
def evaluate(model, loader, criterion):
    model.eval()
    loss_sum = 0.0
    correct = 0
    total = 0

    with torch.no_grad():
        for x, y in loader:
            logits = model(x)
            loss_sum += criterion(logits, y).item()
            predictions = logits.argmax(dim=1)
            correct += (predictions == y).sum().item()
            total += y.size(0)

    return loss_sum / len(loader), correct / total
```

## 3. 개선 실험

| 조정 항목 | 관찰할 현상 | 주의점 |
|---|---|---|
| learning rate | 수렴 속도·진동 | 한 번에 크게 바꾸지 않기 |
| batch size | 속도·기울기 변동 | 메모리 사용량 변화 |
| epochs | 과소·과대 학습 | 검증 추세 함께 보기 |
| normalization | 입력 스케일 안정화 | train/test 동일 적용 |
| Dropout | 과적합 완화 | eval 모드 전환 필수 |
| augmentation | 데이터 다양성 증가 | 라벨 의미 보존 |

## 4. 재현 가능한 비교

한 번에 여러 조건을 모두 바꾸면 어떤 선택이 결과를 개선했는지 알기 어렵습니다. 기준 모델을 두고 한 요소씩 바꾸며 같은 검증 지표로 비교합니다.

```python
torch.manual_seed(42)
```

시드를 고정해도 하드웨어와 연산에 따라 완전한 동일성이 보장되지 않을 수 있지만 실험 변동을 줄이는 출발점입니다.

## 코드로 보기 — 최고 검증 모델 선택

```python
best_val_acc = 0.0
best_state = None

for epoch in range(10):
    train_loss = train_one_epoch(model, train_loader, criterion, optimizer)
    val_loss, val_acc = evaluate(model, val_loader, criterion)

    if val_acc > best_val_acc:
        best_val_acc = val_acc
        best_state = {k: v.cpu().clone() for k, v in model.state_dict().items()}

model.load_state_dict(best_state)
test_loss, test_acc = evaluate(model, test_loader, criterion)
```

## 직접 해보기

1. 학습 손실과 검증 손실이 벌어질 때 가능한 원인을 적어 보세요.
2. 학습률 후보 세 개를 같은 조건에서 비교하는 표를 설계해 보세요.
3. 테스트 세트를 반복해서 보며 설정을 고르면 어떤 문제가 생기는지 설명해 보세요.

<details><summary>정답 보기</summary>

1. 과적합, 데이터 분포 차이, 평가 모드 누락 등을 점검합니다.
2. 학습률 외 배치 크기·에포크·시드를 고정하고 검증 손실과 정확도를 기록합니다.
3. 테스트 데이터에 간접적으로 과적합되어 최종 성능 추정이 낙관적으로 변합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| validation vs test | 설정 선택 vs 최종 평가 |
| loss vs accuracy | 확신까지 반영한 오류 vs 맞힌 비율 |
| 과소적합 vs 과적합 | train도 낮음 vs train만 높음 |
| 모델 개선 vs 평가 누수 | 일반화 향상 vs 평가 데이터에 맞춤 |

## 연결되는 개념

- 이전 글: [MNIST MLP](06-mnist-mlp-classifier.md)
- 용어 확인: [GLOSSARY](GLOSSARY.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 학습·검증·테스트 역할을 구분할 수 있다.
- [ ] 손실과 정확도를 함께 해석할 수 있다.
- [ ] 과적합 신호를 찾을 수 있다.
- [ ] 한 조건씩 바꾸는 실험을 설계할 수 있다.

### 복습 질문 및 답변

**Q1. 학습 정확도가 높으면 테스트 정확도도 반드시 높은가요?**
<details><summary>답</summary>아닙니다. 학습 데이터에 과적합하면 보지 않은 데이터 성능은 낮을 수 있습니다.</details>

**Q2. Dropout 모델 평가 전에 반드시 할 일은 무엇인가요?**
<details><summary>답</summary>`model.eval()`을 호출해 Dropout을 평가 동작으로 전환합니다.</details>

**Q3. 최고 모델을 검증 기준으로 저장하는 이유는 무엇인가요?**
<details><summary>답</summary>마지막 에포크가 항상 가장 일반화 성능이 좋은 시점은 아니기 때문입니다.</details>

## 한 줄 정리

> 모델 개선은 독립된 검증 기준으로 한 조건씩 비교하고 테스트 데이터는 마지막에 지키는 과정입니다.
