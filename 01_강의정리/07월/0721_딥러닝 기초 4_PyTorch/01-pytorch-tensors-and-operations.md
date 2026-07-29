# PyTorch 텐서와 기본 연산

> PyTorch 학습은 데이터를 텐서로 표현하고 모양과 타입을 정확히 읽는 데서 시작합니다.

`Tensor` · `shape` · `dtype` · `reshape` · `device`

## 핵심요약

- 텐서는 여러 차원의 숫자 배열이며 신경망의 입력·가중치·출력을 표현한다.
- `shape`는 각 축의 크기, `dtype`은 원소의 자료형을 알려 준다.
- 텐서 연산은 모양과 타입이 호환되어야 한다.
- `reshape()`와 `flatten()`은 원소 수를 유지하며 모양을 바꾼다.
- 모델과 데이터는 같은 장치에 있어야 연산할 수 있다.

## 1. 텐서 생성과 정보 확인

```python
import torch

x = torch.tensor([[1.0, 2.0], [3.0, 4.0]])

print(x)
print(x.shape)
print(x.dtype)
print(x.ndim)
```

| 정보 | 의미 | 예시 |
|---|---|---|
| `shape` | 축별 크기 | `(2, 2)` |
| `dtype` | 원소 타입 | `torch.float32` |
| `ndim` | 축의 개수 | `2` |
| `numel()` | 전체 원소 수 | `4` |

분류 입력은 보통 실수 텐서, 클래스 라벨은 정수형 `long` 텐서를 사용합니다.

## 2. 인덱싱과 연산

```python
scores = torch.tensor([[0.2, 0.8], [0.7, 0.3]])

print(scores[0])
print(scores[:, 1])
print(scores.mean())
print(scores.argmax(dim=1))
```

`dim`은 어느 축을 따라 계산할지 정합니다. 배치 분류 출력이 `(배치, 클래스)` 모양이면 `argmax(dim=1)`로 샘플별 예측 클래스를 구합니다.

## 3. 모양 바꾸기

```python
images = torch.randn(32, 1, 28, 28)
flat = images.reshape(32, 28 * 28)

print(images.shape)
print(flat.shape)
```

MNIST 이미지는 배치·채널·높이·너비의 4차원 텐서입니다. MLP에 넣기 전 한 이미지의 픽셀을 길이 784의 벡터로 펼칩니다.

## 4. 장치 이동

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
x = x.to(device)
```

GPU를 사용할 때는 모델과 입력, 라벨을 모두 같은 장치로 옮겨야 합니다.

## 코드로 보기 — 미니배치 행렬 연산

```python
batch = torch.tensor([
    [1.0, 2.0],
    [3.0, 4.0],
    [5.0, 6.0],
])
weight = torch.tensor([[0.5], [-0.25]])
bias = torch.tensor([0.1])

logits = batch @ weight + bias
print(logits)
print(logits.shape)
```

### 예상 결과

```text
tensor([[0.1000],
        [0.6000],
        [1.1000]])
torch.Size([3, 1])
```

## 직접 해보기

1. `(2, 3)` 텐서를 만들고 전체 원소 수를 확인해 보세요.
2. `(4, 1, 28, 28)` 이미지를 `(4, 784)`로 바꿔 보세요.
3. `(3, 5)` 점수에서 샘플별 최대 클래스 인덱스를 구해 보세요.

<details><summary>정답 보기</summary>

1. `torch.tensor(...)` 또는 `torch.randn(2, 3)` 뒤 `numel()`을 사용합니다.
2. `images.reshape(images.size(0), -1)`로 배치 축을 유지합니다.
3. `scores.argmax(dim=1)`을 사용합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| shape vs dtype | 텐서 모양 vs 원소 자료형 |
| `dim=0` vs `dim=1` | 첫 축 기준 vs 두 번째 축 기준 |
| reshape vs 데이터 변경 | 모양 변경 vs 원소 값 변경 |
| CPU vs GPU | 실행 장치가 다르며 혼합 연산 불가 |

## 연결되는 개념

- 다음 글: [자동미분과 모델 상태](02-autograd-modules-and-model-modes.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 텐서의 shape, dtype, ndim을 확인할 수 있다.
- [ ] 인덱싱과 축 연산을 사용할 수 있다.
- [ ] 배치 축을 유지하며 이미지를 펼칠 수 있다.
- [ ] 모델과 데이터를 같은 장치에 둘 수 있다.

### 복습 질문 및 답변

**Q1. 분류 라벨이 보통 `long` 타입인 이유는 무엇인가요?**
<details><summary>답</summary>클래스 인덱스를 정수로 표현하고 손실 함수가 그 인덱스를 사용하기 때문입니다.</details>

**Q2. `reshape(32, -1)`에서 `-1`은 무엇인가요?**
<details><summary>답</summary>전체 원소 수에 맞춰 해당 축의 크기를 자동 계산하라는 뜻입니다.</details>

**Q3. 장치 불일치 오류가 나면 무엇을 확인하나요?**
<details><summary>답</summary>모델 파라미터, 입력, 라벨이 모두 같은 CPU 또는 GPU에 있는지 확인합니다.</details>

## 한 줄 정리

> 텐서 코드는 값뿐 아니라 모양·타입·장치를 함께 추적해야 예측 가능합니다.
