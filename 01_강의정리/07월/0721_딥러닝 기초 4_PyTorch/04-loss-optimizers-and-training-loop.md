# 손실 함수·최적화 함수·학습 루프

> 학습은 예측, 손실 계산, 기울기 계산, 파라미터 갱신을 정확한 순서로 반복하는 과정입니다.

`loss` · `optimizer` · `zero_grad()` · `backward()` · `step()`

## 핵심요약

- 손실 함수는 예측과 정답의 차이를 하나의 값으로 만든다.
- Optimizer는 기울기를 이용해 파라미터를 갱신한다.
- 매 배치에서 기울기 초기화 → 순전파 → 손실 → 역전파 → 갱신 순서를 지킨다.
- `CrossEntropyLoss`에는 Softmax 이전 logits와 정수 클래스 라벨을 전달한다.
- 에포크 손실은 배치 손실을 누적한 뒤 기준을 명확히 해 평균 낸다.

## 1. 손실과 최적화 설정

```python
from torch import nn, optim

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

| 구성 요소 | 역할 |
|---|---|
| model | 입력을 예측값으로 변환 |
| criterion | 예측 오류를 손실로 계산 |
| optimizer | 파라미터를 갱신 |
| learning rate | 한 번의 갱신 크기 조절 |

## 2. 한 배치 학습 순서

```python
optimizer.zero_grad()
logits = model(batch_x)
loss = criterion(logits, batch_y)
loss.backward()
optimizer.step()
```

기울기는 기본적으로 누적되므로 `zero_grad()`를 빼면 이전 배치의 기울기가 남습니다.

## 3. 전체 학습 루프

```python
for epoch in range(10):
    model.train()
    running_loss = 0.0

    for batch_x, batch_y in train_loader:
        optimizer.zero_grad()
        logits = model(batch_x)
        loss = criterion(logits, batch_y)
        loss.backward()
        optimizer.step()

        running_loss += loss.item()

    mean_loss = running_loss / len(train_loader)
    print(epoch + 1, mean_loss)
```

## 4. 평가 루프

```python
model.eval()
correct = 0
total = 0

with torch.no_grad():
    for batch_x, batch_y in val_loader:
        logits = model(batch_x)
        predictions = logits.argmax(dim=1)
        correct += (predictions == batch_y).sum().item()
        total += batch_y.size(0)

accuracy = correct / total
```

## 코드로 보기 — 학습과 평가 함수 분리

```python
def train_one_epoch(model, loader, criterion, optimizer):
    model.train()
    total_loss = 0.0

    for x, y in loader:
        optimizer.zero_grad()
        logits = model(x)
        loss = criterion(logits, y)
        loss.backward()
        optimizer.step()
        total_loss += loss.item()

    return total_loss / len(loader)
```

## 직접 해보기

1. 다섯 단계의 배치 학습 순서를 말로 적어 보세요.
2. 검증 루프에서 기울기 기록을 끄는 코드를 작성해 보세요.
3. 맞힌 수와 전체 수로 정확도를 계산해 보세요.

<details><summary>정답 보기</summary>

1. 초기화, 순전파, 손실 계산, 역전파, 파라미터 갱신 순서입니다.
2. `model.eval()`과 `with torch.no_grad():`를 사용합니다.
3. `(pred == y).sum().item()`을 누적하고 전체 샘플 수로 나눕니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| loss vs accuracy | 학습 가능한 연속 오류값 vs 맞힌 비율 |
| `backward()` vs `step()` | 기울기 계산 vs 파라미터 갱신 |
| logits vs probabilities | 활성화 전 점수 vs 확률 해석값 |
| batch 평균 vs sample 평균 | 배치별 동일 가중 vs 샘플별 동일 가중 |

## 연결되는 개념

- 이전 글: [Dataset과 DataLoader](03-datasets-dataloaders-and-splits.md)
- 다음 글: [이진분류 워크숍](05-binary-classification-workshop.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 손실 함수와 Optimizer의 역할을 설명할 수 있다.
- [ ] 배치 학습의 다섯 단계를 순서대로 작성할 수 있다.
- [ ] 학습 루프와 평가 루프를 구분할 수 있다.
- [ ] 손실과 정확도를 누적할 수 있다.

### 복습 질문 및 답변

**Q1. `zero_grad()`를 매 배치 호출하는 이유는 무엇인가요?**
<details><summary>답</summary>이전 배치에서 계산한 기울기가 기본적으로 누적되기 때문입니다.</details>

**Q2. CrossEntropyLoss 앞에서 Softmax를 생략하는 이유는 무엇인가요?**
<details><summary>답</summary>손실 함수가 수치적으로 안정적인 방식으로 내부에서 해당 계산을 포함하기 때문입니다.</details>

**Q3. 평가에서 `optimizer.step()`을 호출하지 않는 이유는 무엇인가요?**
<details><summary>답</summary>검증과 테스트는 파라미터를 바꾸지 않고 현재 모델의 성능을 측정하는 단계이기 때문입니다.</details>

## 한 줄 정리

> PyTorch 학습 루프는 기울기 초기화부터 파라미터 갱신까지의 순서가 정확해야 합니다.
