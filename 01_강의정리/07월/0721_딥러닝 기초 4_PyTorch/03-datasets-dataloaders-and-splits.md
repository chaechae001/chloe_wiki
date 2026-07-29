# Dataset과 DataLoader

> 좋은 학습 루프는 데이터를 역할별로 나누고 같은 모양의 미니배치로 공급합니다.

`Dataset` · `TensorDataset` · `DataLoader` · `batch_size` · `shuffle`

## 핵심요약

- Dataset은 개별 샘플과 라벨을 제공한다.
- DataLoader는 Dataset을 미니배치로 묶어 반복 가능하게 만든다.
- 학습·검증·테스트 데이터는 목적이 다르므로 분리한다.
- 학습 데이터는 보통 섞고 평가 데이터는 순서를 유지한다.
- 입력과 라벨의 길이·타입·모양을 먼저 검증한다.

## 1. TensorDataset 구성

```python
import torch
from torch.utils.data import TensorDataset, DataLoader

features = torch.randn(100, 2)
labels = torch.randint(0, 2, (100,), dtype=torch.long)

dataset = TensorDataset(features, labels)
print(len(dataset))
```

`features[i]`와 `labels[i]`가 하나의 학습 샘플을 이룹니다. 두 텐서의 첫 축 길이는 같아야 합니다.

## 2. 미니배치 만들기

```python
loader = DataLoader(
    dataset,
    batch_size=32,
    shuffle=True,
)

batch_x, batch_y = next(iter(loader))
print(batch_x.shape, batch_y.shape)
```

배치 크기 32라면 입력 모양은 `(32, 2)`, 라벨 모양은 `(32,)`입니다. 마지막 배치는 데이터 수에 따라 더 작을 수 있습니다.

## 3. 데이터 역할 분리

| 구분 | 목적 | 학습에 사용? | 보통 shuffle? |
|---|---|---:|---:|
| train | 파라미터 업데이트 | 예 | 예 |
| validation | 설정 선택·과적합 관찰 | 아니요 | 아니요 |
| test | 최종 일반화 성능 측정 | 아니요 | 아니요 |

검증 데이터로 반복해서 모델과 설정을 선택한 뒤 테스트 데이터는 마지막 확인에 사용합니다.

## 4. MNIST 분할

```python
from torch.utils.data import random_split

train_set, val_set = random_split(full_train_set, [50000, 10000])
train_loader = DataLoader(train_set, batch_size=32, shuffle=True)
val_loader = DataLoader(val_set, batch_size=32, shuffle=False)
test_loader = DataLoader(test_set, batch_size=32, shuffle=False)
```

## 코드로 보기 — 배치 점검 함수

```python
def inspect_loader(loader):
    batch_x, batch_y = next(iter(loader))
    print("inputs:", batch_x.shape, batch_x.dtype)
    print("labels:", batch_y.shape, batch_y.dtype)
    print("batches:", len(loader))

inspect_loader(loader)
```

## 직접 해보기

1. 샘플 65개를 배치 32로 구성했을 때 배치 수를 확인해 보세요.
2. 학습용과 테스트용 DataLoader의 shuffle 값을 다르게 설정해 보세요.
3. 첫 배치의 입력·라벨 모양과 타입을 출력해 보세요.

<details><summary>정답 보기</summary>

1. 마지막 작은 배치를 포함하면 3개입니다.
2. 학습은 `True`, 테스트는 `False`로 둡니다.
3. `next(iter(loader))`로 배치를 꺼내 `.shape`, `.dtype`을 확인합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| Dataset vs DataLoader | 샘플 제공 규칙 vs 배치 반복기 |
| batch size vs 데이터 수 | 한 번에 처리할 수 vs 전체 샘플 수 |
| validation vs test | 설정 선택용 vs 최종 평가용 |
| shuffle vs split | 순서 섞기 vs 데이터 역할 나누기 |

## 연결되는 개념

- 이전 글: [자동미분과 신경망 모듈](02-autograd-modules-and-model-modes.md)
- 다음 글: [손실·최적화·학습 루프](04-loss-optimizers-and-training-loop.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] Dataset과 DataLoader의 역할을 구분할 수 있다.
- [ ] 입력과 라벨을 TensorDataset으로 묶을 수 있다.
- [ ] 학습·검증·테스트 데이터를 나눌 수 있다.
- [ ] 첫 배치 모양과 타입을 점검할 수 있다.

### 복습 질문 및 답변

**Q1. 학습 데이터를 섞는 이유는 무엇인가요?**
<details><summary>답</summary>매 에포크의 배치 구성을 바꿔 데이터 순서에 대한 편향을 줄이기 위해서입니다.</details>

**Q2. 테스트 데이터로 파라미터를 업데이트하면 안 되는 이유는 무엇인가요?**
<details><summary>답</summary>최종 성능을 독립적으로 측정할 기준이 사라지기 때문입니다.</details>

**Q3. 마지막 배치가 작은 것은 오류인가요?**
<details><summary>답</summary>아닙니다. 전체 데이터 수가 배치 크기로 나누어떨어지지 않으면 정상적으로 발생합니다.</details>

## 한 줄 정리

> DataLoader는 역할별 Dataset을 반복 가능한 미니배치로 바꾸어 학습 루프와 연결합니다.
