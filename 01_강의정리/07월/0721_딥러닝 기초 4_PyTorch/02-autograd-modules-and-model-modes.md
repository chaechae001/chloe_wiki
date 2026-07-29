# 자동미분과 신경망 모듈

> PyTorch는 순전파 연산 기록을 이용해 기울기를 계산하고 모델 파라미터를 학습합니다.

`autograd` · `nn.Module` · `forward` · `train()` · `eval()`

## 핵심요약

- `requires_grad=True`인 텐서 연산은 계산 그래프에 기록된다.
- `backward()`는 손실에서 각 파라미터까지의 기울기를 계산한다.
- 모델은 `nn.Module`을 상속하고 `forward()`에 순전파를 정의한다.
- `train()`과 `eval()`은 Dropout 등 상태 의존 계층의 동작을 전환한다.
- 평가에서는 `torch.no_grad()`로 기울기 기록을 끈다.

## 1. 자동미분

```python
import torch

w = torch.tensor(2.0, requires_grad=True)
x = torch.tensor(3.0)
y = w * x
loss = (y - 10) ** 2

loss.backward()
print(w.grad)
```

`w.grad`에는 손실을 줄이기 위해 `w`를 어느 방향으로 얼마나 바꿔야 하는지 나타내는 미분값이 저장됩니다.

## 2. `nn.Module` 모델

```python
from torch import nn

class BinaryMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(2, 16)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(16, 2)

    def forward(self, x):
        x = self.fc1(x)
        x = self.relu(x)
        return self.fc2(x)
```

`__init__()`에는 학습할 계층을 등록하고 `forward()`에는 데이터가 통과하는 순서를 작성합니다. 모델을 `model(x)`로 호출하면 `forward()`가 실행됩니다.

## 3. 학습·평가 모드

```python
model.train()
# 학습 루프

model.eval()
with torch.no_grad():
    # 검증 또는 테스트
    pass
```

`eval()`은 기울기를 끄는 명령이 아닙니다. 모델 상태는 `eval()`, 기울기 기록은 `torch.no_grad()`가 각각 담당합니다.

## 코드로 보기 — 파라미터와 출력 모양 확인

```python
model = BinaryMLP()
sample = torch.randn(8, 2)
logits = model(sample)

print(logits.shape)
for name, parameter in model.named_parameters():
    print(name, parameter.shape)
```

### 예상 핵심 결과

```text
torch.Size([8, 2])
fc1.weight torch.Size([16, 2])
fc1.bias torch.Size([16])
fc2.weight torch.Size([2, 16])
fc2.bias torch.Size([2])
```

## 직접 해보기

1. 입력 4개, 은닉 8개, 출력 3개인 MLP를 정의해 보세요.
2. 모델의 모든 파라미터 이름과 모양을 출력해 보세요.
3. 평가 모드와 `no_grad()`를 함께 사용하는 코드를 작성해 보세요.

<details><summary>정답 보기</summary>

1. `nn.Linear(4, 8)`과 `nn.Linear(8, 3)`을 순서대로 사용합니다.
2. `model.named_parameters()`를 반복합니다.
3. `model.eval()` 뒤 `with torch.no_grad():` 블록에서 추론합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 순전파 vs 역전파 | 예측 계산 vs 기울기 계산 |
| `train()` vs `eval()` | 학습 동작 모드 vs 평가 동작 모드 |
| `eval()` vs `no_grad()` | 계층 상태 전환 vs 계산 그래프 기록 중단 |
| 계층 정의 vs 실행 순서 | `__init__()` vs `forward()` |

## 연결되는 개념

- 이전 글: [텐서와 기본 연산](01-pytorch-tensors-and-operations.md)
- 다음 글: [Dataset과 DataLoader](03-datasets-dataloaders-and-splits.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 자동미분의 목적을 설명할 수 있다.
- [ ] `nn.Module`을 상속해 모델을 만들 수 있다.
- [ ] 계층 정의와 순전파를 나눌 수 있다.
- [ ] 학습과 평가 모드를 올바르게 전환할 수 있다.

### 복습 질문 및 답변

**Q1. `backward()` 결과는 어디에 저장되나요?**
<details><summary>답</summary>기울기가 필요한 각 파라미터의 `.grad`에 누적됩니다.</details>

**Q2. 출력층 뒤에 항상 Softmax를 넣어야 하나요?**
<details><summary>답</summary>아닙니다. `CrossEntropyLoss`를 쓸 때는 원시 logits를 전달합니다.</details>

**Q3. 평가 전에 `model.eval()`이 필요한 이유는 무엇인가요?**
<details><summary>답</summary>Dropout과 BatchNorm처럼 학습·평가 시 동작이 다른 계층을 평가 상태로 바꾸기 위해서입니다.</details>

## 한 줄 정리

> `nn.Module`은 순전파 구조를 담고 자동미분은 손실에서 파라미터까지 학습 신호를 전달합니다.
