# MNIST MLP 분류 모델

> 28×28 손글씨 이미지를 펼쳐 10개 숫자 클래스로 분류하는 MLP를 구현합니다.

`MNIST` · `ToTensor` · `flatten` · `MLP` · `10-class classification`

## 핵심요약

- MNIST 이미지는 1×28×28 텐서이며 라벨은 0~9의 정수다.
- `ToTensor()`로 이미지를 텐서로 변환한다.
- MLP 입력 전에 이미지를 784차원 벡터로 펼친다.
- 784 → 128 → 64 → 10 구조와 ReLU를 사용할 수 있다.
- 학습·검증·테스트 루프의 구조는 이진분류와 동일하고 차원만 달라진다.

## 1. 데이터 준비

```python
from torchvision import datasets, transforms

transform = transforms.ToTensor()
train_dataset = datasets.MNIST(
    root="data",
    train=True,
    download=True,
    transform=transform,
)
test_dataset = datasets.MNIST(
    root="data",
    train=False,
    download=True,
    transform=transform,
)
```

## 2. 학습·검증 분할

```python
train_dataset, val_dataset = torch.utils.data.random_split(
    train_dataset,
    [50000, 10000],
)

train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
val_loader = DataLoader(val_dataset, batch_size=32, shuffle=False)
test_loader = DataLoader(test_dataset, batch_size=32, shuffle=False)
```

## 3. MLP 구조

```python
class MNISTMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(28 * 28, 128)
        self.fc2 = nn.Linear(128, 64)
        self.fc3 = nn.Linear(64, 10)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = x.reshape(x.size(0), -1)
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        return self.fc3(x)
```

## 코드로 보기 — 한 배치 확인

```python
model = MNISTMLP()
images, labels = next(iter(train_loader))
logits = model(images)
predictions = logits.argmax(dim=1)

print(images.shape)
print(logits.shape)
print(labels[:8])
print(predictions[:8])
```

### 예상 모양

```text
images: torch.Size([32, 1, 28, 28])
logits: torch.Size([32, 10])
```

## 직접 해보기

1. 이미지 한 장의 전체 픽셀 수를 계산해 보세요.
2. 출력층 차원을 10으로 설정하는 이유를 설명해 보세요.
3. 검증 데이터 10,000개를 분리한 뒤 학습 데이터 수를 확인해 보세요.

<details><summary>정답 보기</summary>

1. `28 * 28 = 784`입니다.
2. 숫자 0부터 9까지 10개 클래스의 logits가 필요하기 때문입니다.
3. 원래 학습 데이터 60,000개 중 50,000개가 남습니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 이미지 shape vs MLP 입력 | `(N,1,28,28)` vs `(N,784)` |
| 라벨 9 vs 클래스 수 10 | 최대 인덱스 vs 전체 클래스 개수 |
| Softmax 출력 vs logits | 확률 vs 손실 함수에 전달할 원시 점수 |
| train split vs test set | 학습 내부 분할 vs 독립 최종 데이터 |

## 연결되는 개념

- 이전 글: [이진분류 워크숍](05-binary-classification-workshop.md)
- 다음 글: [평가와 성능 개선](07-evaluation-and-improvement.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] MNIST 데이터셋을 텐서로 불러올 수 있다.
- [ ] 이미지를 배치별로 펼칠 수 있다.
- [ ] 3개 Linear 계층의 차원을 연결할 수 있다.
- [ ] 10개 클래스 예측을 구할 수 있다.

### 복습 질문 및 답변

**Q1. 배치 크기를 코드에 고정해 reshape하면 어떤 문제가 생기나요?**
<details><summary>답</summary>마지막 배치가 더 작을 때 모양 오류가 나므로 `x.size(0)`을 사용해야 합니다.</details>

**Q2. ReLU는 어디에 적용하나요?**
<details><summary>답</summary>은닉 Linear 계층 뒤에 적용하고 CrossEntropyLoss를 쓰는 출력층 뒤에는 두지 않습니다.</details>

**Q3. 검증 데이터가 필요한 이유는 무엇인가요?**
<details><summary>답</summary>테스트 데이터를 보지 않고 학습 진행과 설정 선택을 판단하기 위해서입니다.</details>

## 한 줄 정리

> MNIST MLP는 이미지를 784차원으로 펼치고 은닉 표현을 거쳐 10개 logits로 변환합니다.
