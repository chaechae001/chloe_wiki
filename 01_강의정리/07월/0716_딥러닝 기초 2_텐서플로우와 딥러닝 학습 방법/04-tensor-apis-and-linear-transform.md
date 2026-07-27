# 텐서 API와 선형 변환 — 숫자를 만들고 가중치로 조합하는 법

> 신경망의 화려한 층도 안쪽을 들여다보면 텐서를 만들고, 바꿀 값은 갱신하고, $Wx+b$로 입력을 조합하는 과정입니다. 기본 API와 shape 규칙을 익히면 모델 한 층이 무엇을 계산하는지 손으로도 검증할 수 있습니다.

`constant` · `linspace` · `range` · `Variable.assign` · `Wx+b`

## 핵심요약

- `tf.constant`, `tf.zeros`, `tf.ones`는 바뀌지 않는 기본 텐서를 만든다.
- `tf.linspace`는 양 끝을 포함해 개수를 맞추고, `tf.range`는 끝값 미만까지 간격을 맞춘다.
- 학습 중 바뀌는 가중치와 편향은 `tf.Variable`로 만들고 `assign`으로 값을 갱신한다.
- 선형 변환 $Wx+b$에서는 행렬 곱이 가능한 안쪽 차원과 편향의 브로드캐스팅을 확인해야 한다.
- 저장된 실행에서 시퀀스와 변수 갱신 값이 확인됐고, $W=[[1,-1]]$, $x=[1,0]$, $b=[0.5]$의 결과는 정확히 `1.5`였다.

## 1. 목적에 맞는 텐서 생성 API

### 1) 정의

TensorFlow는 값을 직접 적는 방법뿐 아니라 특정 모양과 규칙을 가진 텐서를 만드는 함수를 제공합니다.
| API | 만드는 값 | 대표 용도 |
|---|---|---|
| `tf.constant` | 값이 고정된 텐서 | 입력 샘플, 기준값 |
| `tf.zeros` | 모든 값이 0인 텐서 | 초기 버퍼, 마스크 |
| `tf.ones` | 모든 값이 1인 텐서 | 기본 가중 마스크 |
| `tf.linspace` | 양 끝을 포함한 균등 간격 | 임계값·좌표 샘플 |
| `tf.range` | 끝값 미만의 일정 간격 | 인덱스·반복 구간 |
| `tf.Variable` | 갱신할 수 있는 텐서 | 가중치·편향 |

### 2) 왜 필요한가

데이터 준비와 모델 학습에서는 같은 모양의 숫자 묶음을 반복해서 만들어야 합니다. 리스트를 일일이 작성하면 길이와 dtype을 틀리기 쉽지만, 목적에 맞는 API를 쓰면 의도가 코드에 드러납니다.

예를 들어 `tf.zeros((2, 3))`만 보아도 2행 3열의 빈 초기 상태라는 뜻을 바로 읽을 수 있습니다.

### 3) 핵심 흐름 재구성

상수 텐서는 계산의 재료이고, 변수 텐서는 학습 과정에서 조정되는 손잡이입니다.

```python
import tensorflow as tf

fixed = tf.constant([1.0, 2.0, 3.0])
weights = tf.Variable([0.1, -0.2, 0.3])

weights.assign([1.0, 2.0, 3.0])
```

`fixed`는 새 텐서를 다시 만들지 않는 한 값을 바꾸지 않습니다. `weights`는 변수 객체의 정체성을 유지한 채 내부 값을 교체할 수 있습니다.

이는 최적화 알고리즘이 한 단계마다 가중치를 갱신하는 방식과 연결됩니다.

$$
W_{t+1} \leftarrow W_t - \alpha \nabla L(W_t)
$$

쉽게 말하면 현재 가중치에서 손실이 작아지는 방향으로 조금 이동한 값을 같은 변수에 다시 저장한다는 뜻입니다.

### 4) 쉬운 예시

요리 레시피에 비유하면 상수 텐서는 오븐 온도처럼 이번 계산에서 고정해 둔 기준입니다. 변수 텐서는 간을 보며 조절하는 소금 양과 같습니다.

첫 시도에서 짜면 값을 줄이고, 싱거우면 값을 늘리지만 여전히 같은 '소금 양' 항목을 조정합니다.

`assign`은 새 레시피 항목을 만드는 것이 아니라 그 항목의 현재 값을 덮어쓰는 동작입니다.
### 5) 코드 예시

```python
import tensorflow as tf

base = tf.constant([1, 2, 3], dtype=tf.float32)
zero_grid = tf.zeros((2, 3), dtype=tf.float32)
one_grid = tf.ones((2, 3), dtype=tf.float32)

points = tf.linspace(0.0, 1.0, 5)
indices = tf.range(0, 10, 2)

weights = tf.Variable([0.1, -0.2, 0.3], dtype=tf.float32)
before = weights.numpy().copy()
weights.assign([1.0, 2.0, 3.0])

print("constant:", base.numpy())
print("linspace:", points.numpy())
print("range:", indices.numpy())
print("before:", before)
print("after:", weights.numpy())
```

```text
constant: [1. 2. 3.]
linspace: [0.   0.25 0.5  0.75 1.  ]
range: [0 2 4 6 8]
before: [ 0.1 -0.2  0.3]
after: [1. 2. 3.]
```

저장된 출력은 `linspace`가 양 끝 0과 1을 포함하고, `range`는 끝값 10을 포함하지 않는 차이를 보여 줍니다.

### 6) 헷갈리는 점

`tf.Variable`을 썼다고 값이 저절로 좋아지는 것은 아닙니다. 손실과 기울기를 계산한 뒤 최적화 단계가 변수 값을 실제로 갱신해야 학습이 일어납니다.

또한 `assign`에 넣는 값은 원래 변수와 shape가 맞아야 합니다.

`tf.linspace(0.0, 1.0, 5)`의 세 번째 인자는 간격이 아니라 **생성할 값의 개수**입니다.

반대로 `tf.range(0, 10, 2)`의 세 번째 인자는 값 사이의 **증가 간격**입니다.

### 7) 한 줄 정리

> 고정 재료는 상수 텐서로 만들고, 학습하며 바꿀 파라미터는 변수 텐서로 만들어 갱신한다.

## 2. 선형 변환 $Wx+b$

### 1) 정의

선형 변환은 입력 특성에 가중치를 곱해 더하고 편향을 더하는 계산입니다.
$$
f(x) = Wx + b
$$

$x$는 입력, $W$는 각 입력을 얼마나 반영할지 정하는 가중치, $b$는 전체 결과를 이동시키는 편향입니다.

신경망의 Dense 층도 활성화 함수에 들어가기 전에는 이 계산을 수행합니다.
### 2) 왜 필요한가

입력 특성은 예측에 같은 정도로 중요하지 않습니다. 가중치는 각 특성의 영향력을 조절하고, 편향은 입력이 모두 0이어도 기준 출력을 만들 수 있게 합니다.

따라서 $Wx+b$는 여러 입력을 하나 이상의 출력으로 조합하는 신경망의 가장 기본적인 계산 단위입니다.

### 3) 핵심 흐름 재구성

다음과 같은 shape를 생각해 봅시다.
$$
W: (1,2), \qquad x: (2,1), \qquad b: (1,)
$$

행렬 곱의 안쪽 차원 2가 서로 같으므로 계산할 수 있습니다.

$$
(1,2) @ (2,1) \rightarrow (1,1)
$$

구체적인 값은 다음과 같습니다.

$$
W = \begin{bmatrix}1 & -1\end{bmatrix}, \quad
x = \begin{bmatrix}1\\0\end{bmatrix}, \quad
b = \begin{bmatrix}0.5\end{bmatrix}
$$

$$
Wx+b = (1 \times 1) + (-1 \times 0) + 0.5 = 1.5
$$

첫 번째 입력은 가중치 1만큼 반영되고, 두 번째 입력은 값이 0이라 가중치가 -1이어도 이번 출력에는 영향을 주지 않습니다.

### 4) 쉬운 예시

카페의 음료 가격을 계산한다고 생각해 봅시다. 입력 $x$는 샷과 시럽의 개수, $W$는 각 추가 항목이 가격에 미치는 영향, $b$는 기본 음료 가격입니다.

추가 항목이 하나도 없어도 기본 가격은 남으므로 편향이 필요합니다.

여러 주문을 한꺼번에 계산할 때는 주문표를 행렬로 묶어 같은 가중치 계산을 반복합니다.
### 5) 코드 예시

```python
import tensorflow as tf

W = tf.Variable([[1.0, -1.0]], dtype=tf.float32)
b = tf.Variable([0.5], dtype=tf.float32)
x = tf.constant([[1.0], [0.0]], dtype=tf.float32)

def linear_forward(inputs):
    return tf.matmul(W, inputs) + b

result = linear_forward(x)
print("W shape:", W.shape)
print("x shape:", x.shape)
print("result shape:", result.shape)
print("result:", result.numpy().item())
```

```text
W shape: (1, 2)
x shape: (2, 1)
result shape: (1, 1)
result: 1.5
```

저장된 실행에서도 $W$, $x$, $b$가 각각 `[[1, -1]]`, `[1, 0]`, `[0.5]`로 확인됐고 최종 출력은 `1.5`였습니다.

### 6) 헷갈리는 점

수식의 $Wx$는 성분별 곱이 아니라 행렬 곱입니다. 또한 $b$의 shape `(1,)`과 행렬 곱 결과 `(1, 1)`이 완전히 같지 않아도 브로드캐스팅 규칙에 따라 덧셈이 가능합니다.

브로드캐스팅은 작은 텐서를 필요한 방향으로 확장해 계산하는 규칙이지, 실제 데이터를 무조건 복제해 저장한다는 뜻은 아닙니다. 입력을 한 건씩 열 벡터로 둘지, 여러 건을 행으로 쌓을지에 따라 $W$와 $x$의 배치 방향이 달라질 수 있습니다.

중요한 것은 외운 배치가 아니라 행렬 곱의 안쪽 차원이 맞고 결과 축의 의미가 분명한지 확인하는 것입니다.

### 7) 한 줄 정리

> $Wx+b$는 입력을 가중치로 조합하고 편향으로 기준점을 더해 한 층의 출력을 만드는 계산이다.

## 코드로 보기 — 생성 API부터 선형 출력까지

```python
import tensorflow as tf

grid = tf.ones((1, 2), dtype=tf.float32)
steps = tf.linspace(0.0, 1.0, 5)
index = tf.range(0, 10, 2)
weight = tf.Variable([[1.0, -1.0]], dtype=tf.float32)
bias = tf.Variable([0.5], dtype=tf.float32)
sample = tf.constant([[1.0], [0.0]], dtype=tf.float32)

output = tf.matmul(weight, sample) + bias

print("grid:", grid.numpy())
print("steps:", steps.numpy())
print("index:", index.numpy())
print("output:", output.numpy().item())
```

### 코드 목적

자주 쓰는 생성 API로 텐서를 준비하고, 변수 가중치와 상수 입력을 $Wx+b$로 결합합니다.

### 코드 흐름

1. `ones`, `linspace`, `range`로 서로 다른 목적의 텐서를 만듭니다.
2. 가중치와 편향은 이후 갱신할 수 있도록 변수로 만듭니다.
3. 입력은 이번 계산에서 고정하므로 상수로 만듭니다.
4. 행렬 곱과 편향 덧셈을 수행하고 스칼라 값을 확인합니다.

### 실행 결과 해석

저장된 출력에서 `linspace`는 `[0, 0.25, 0.5, 0.75, 1]`, `range`는 `[0, 2, 4, 6, 8]`이었습니다. 변수는 `[0.1, -0.2, 0.3]`에서 `[1, 2, 3]`으로 갱신되어 `assign`의 덮어쓰기 동작을 보여 줬습니다.

선형 변환은 $(1 \times 1) + (-1 \times 0) + 0.5$로 계산되어 `1.5`가 됐습니다.

### 실무 연결

`zeros`와 `ones`는 마스크·초기 상태·테스트 입력을 만들 때 자주 사용합니다.

`range`는 데이터 인덱스를 만들 때, `linspace`는 일정 구간의 임계값이나 좌표를 고르게 시험할 때 유용합니다.

$Wx+b$는 회귀 모델뿐 아니라 이미지·문장·표 데이터의 특성을 다음 층 표현으로 바꾸는 Dense 층의 중심 계산입니다.

## 직접 해보기

1. `tf.linspace(0.0, 2.0, 5)`와 `tf.range(0, 5, 1)`이 만드는 값을 각각 적어 보세요.
2. $W=[2,-1]$, $x=[3,4]^T$, $b=0.5$일 때 $Wx+b$를 계산해 보세요.
3. 학습률에 따라 계속 바뀌어야 하는 가중치를 `tf.constant`로 만들었다면 어떤 API로 바꾸는 것이 적절한가요?

<details>
<summary>정답 보기</summary>

1. `linspace`는 `[0, 0.5, 1, 1.5, 2]`, `range`는 `[0, 1, 2, 3, 4]`입니다. 앞 함수는 끝값을 포함해 개수를 맞추고, 뒤 함수는 끝값 미만까지 간격을 적용합니다.
2. $2 \times 3 + (-1) \times 4 + 0.5 = 2.5$입니다.
3. `tf.Variable`로 만들고 `assign` 또는 최적화 단계로 값을 갱신해야 합니다. 상수 텐서는 학습 파라미터처럼 내부 값을 바꾸는 용도에 맞지 않습니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| `tf.constant` vs `tf.Variable` | 고정 계산 재료 vs 학습 중 갱신할 상태 |
| `tf.linspace` vs `tf.range` | 끝을 포함해 개수 지정 vs 끝 미만까지 간격 지정 |
| `assign` vs `=` | 변수 내부 값 갱신 vs 파이썬 이름이 가리키는 객체 변경 |
| 성분별 곱 vs 행렬 곱 | 같은 위치끼리 곱하기 vs 행과 열을 내적하기 |
| 편향 덧셈 vs 가중치 곱 | 출력 기준점 이동 vs 입력 특성 영향력 조절 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [텐서와 즉시 실행](03-tensors-and-eager-execution.md)
- 다음에 이어지는 개념: [에포크·배치·반복과 데이터 파이프라인](05-epoch-batch-iteration-and-tfdata.md)
- 함께 보면 좋은 키워드: `dtype`, `broadcasting`, `Dense layer`

## 셀프 체크

- [ ] 상수 텐서와 변수 텐서의 용도를 구분할 수 있다.
- [ ] `linspace`와 `range`의 끝값 처리 차이를 설명할 수 있다.
- [ ] `assign`이 필요한 이유를 학습 과정과 연결할 수 있다.
- [ ] $Wx+b$를 작은 숫자로 직접 계산할 수 있다.
- [ ] 행렬 곱 전후의 shape를 확인할 수 있다.

### 복습 질문 및 답변

**Q1. `tf.linspace(0.0, 1.0, 5)`의 간격은 왜 0.2가 아니라 0.25인가요?**

<details>
<summary>답</summary>

0과 1을 포함한 값 다섯 개 사이에는 구간이 네 개뿐입니다. 따라서 각 간격은 $(1-0)/(5-1)=0.25$입니다.

</details>

**Q2. 변수 갱신 전 값을 따로 보관할 때 `.numpy().copy()`를 쓴 이유는 무엇인가요?**

<details>
<summary>답</summary>

갱신 전 시점의 숫자를 독립된 배열로 남겨 두기 위해서입니다. 이렇게 하면 이후 `assign`으로 변수 값이 바뀌어도 비교용 값은 그대로 유지됩니다.

</details>

**Q3. $W$의 shape가 `(1, 2)`이고 $x$의 shape가 `(3, 1)`이면 왜 곱할 수 없나요?**

<details>
<summary>답</summary>

행렬 곱에서는 왼쪽 행렬의 열 수와 오른쪽 행렬의 행 수가 같아야 합니다. 여기서는 2와 3이 달라 어떤 입력 두 개에 가중치 두 개를 대응시킬지 정할 수 없습니다.

</details>

## 한 줄 정리

> 텐서 생성 API는 계산 재료의 값과 모양을 분명하게 만들고, $Wx+b$는 그 재료를 학습 가능한 가중치로 조합해 다음 표현으로 바꾼다.
