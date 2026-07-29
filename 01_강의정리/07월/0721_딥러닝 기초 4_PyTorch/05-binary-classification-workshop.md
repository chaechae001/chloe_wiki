# PyTorch 이진분류 워크숍

> 두 개의 입력 특성에서 두 클래스를 구분하는 실습으로 전체 분류 파이프라인을 연결합니다.

`binary classification` · `MLP` · `logits` · `CrossEntropyLoss` · `accuracy`

## 핵심요약

- 특성 2개와 클래스 라벨을 텐서로 만들고 역할별로 분할한다.
- 입력 2 → 은닉 16 → 은닉 8 → 출력 2의 MLP를 구성할 수 있다.
- 출력층은 클래스별 logits 두 개를 반환한다.
- 학습 데이터만 섞고 검증·테스트는 평가용으로 유지한다.
- 테스트 손실과 정확도는 전체 배치를 누적해 계산한다.

## 1. 데이터 파이프라인

```python
X = torch.tensor(frame[["x1", "x2"]].values, dtype=torch.float32)
y = torch.tensor(frame["label"].values, dtype=torch.long)
```

입력은 `(샘플 수, 2)`, 라벨은 `(샘플 수,)` 모양입니다. 분할 전에 두 모양과 클래스별 개수를 확인합니다.

## 2. 모델 정의

```python
class BinaryClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(2, 16),
            nn.ReLU(),
            nn.Linear(16, 8),
            nn.ReLU(),
            nn.Linear(8, 2),
        )

    def forward(self, x):
        return self.network(x)
```

두 출력은 클래스 0과 1에 대한 점수입니다. 예측 클래스는 `argmax(dim=1)`로 구합니다.

## 3. 학습 구성

```python
model = BinaryClassifier()
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
```

학습률은 한 번의 파라미터 갱신 크기를 조절합니다. 너무 크면 불안정하고 너무 작으면 학습이 느릴 수 있습니다.

## 코드로 보기 — 테스트 평가

```python
model.eval()
test_loss = 0.0
correct = 0
total = 0

with torch.no_grad():
    for x, y in test_loader:
        logits = model(x)
        loss = criterion(logits, y)
        test_loss += loss.item()

        predictions = logits.argmax(dim=1)
        correct += (predictions == y).sum().item()
        total += y.size(0)

test_loss /= len(test_loader)
test_acc = correct / total
```

## 직접 해보기

1. 첫 배치가 모델을 통과한 출력 모양을 예측해 보세요.
2. 은닉층을 하나 제거했을 때 계층 연결 차원을 조정해 보세요.
3. 테스트 정확도 계산에서 전체 수를 배치 수로 나누면 왜 틀리는지 설명해 보세요.

<details><summary>정답 보기</summary>

1. 배치가 32라면 `(32, 2)`입니다.
2. 앞 계층 출력 차원과 다음 계층 입력 차원을 같게 맞춥니다.
3. 정확도는 맞힌 샘플 수를 전체 샘플 수로 나누어야 하기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 이진분류 vs 출력 하나 | 이 실습은 두 클래스 logits를 출력 |
| 라벨 shape vs 출력 shape | `(N,)` vs `(N, 2)` |
| 데이터 노이즈 vs 코드 오류 | 예외 샘플 존재 vs 구현 실수 |
| train accuracy vs test accuracy | 학습 데이터 적합도 vs 일반화 측정 |

## 연결되는 개념

- 이전 글: [학습 루프](04-loss-optimizers-and-training-loop.md)
- 다음 글: [MNIST MLP](06-mnist-mlp-classifier.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 2차원 특성과 클래스 라벨을 텐서로 만들 수 있다.
- [ ] 계층의 입출력 차원을 연결할 수 있다.
- [ ] CrossEntropyLoss에 맞는 출력과 라벨을 준비할 수 있다.
- [ ] 테스트 손실과 정확도를 계산할 수 있다.

### 복습 질문 및 답변

**Q1. 마지막 계층의 출력 차원이 2인 이유는 무엇인가요?**
<details><summary>답</summary>클래스 0과 클래스 1 각각에 대한 점수를 출력하기 때문입니다.</details>

**Q2. 평가 DataLoader에서 shuffle이 필수가 아닌 이유는 무엇인가요?**
<details><summary>답</summary>파라미터를 갱신하지 않고 전체 성능을 집계하므로 순서가 결과에 영향을 주지 않기 때문입니다.</details>

**Q3. 라벨 노이즈가 있으면 정확도가 항상 100%가 되지 않을 수 있나요?**
<details><summary>답</summary>네. 특성과 일치하지 않는 라벨은 모델이 일반 패턴을 학습해도 오답으로 집계될 수 있습니다.</details>

## 한 줄 정리

> 이진분류 파이프라인은 데이터 모양, 출력 클래스 수, 손실 함수와 평가 계산이 서로 맞아야 완성됩니다.
