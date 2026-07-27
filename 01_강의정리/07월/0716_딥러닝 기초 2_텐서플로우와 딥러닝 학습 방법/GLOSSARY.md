# 용어집

이번 회차에 등장한 핵심 용어를 쉬운 말로 정리했습니다. 학습 원리, 텐서 연산, 데이터 파이프라인, Keras 분류 흐름으로 나눠 두었으니 막히는 용어를 빠르게 찾아보세요.

## 손실·최적화와 자동미분

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 손실 함수(Loss Function) | 정답과 예측의 차이를 하나의 숫자로 나타내는 함수. 모델 학습은 이 값을 작게 만드는 파라미터를 찾는 과정 | [01](01-loss-and-gradient-descent.md) | MSE, BCE, 최적화 |
| 평균제곱오차(MSE) | 각 예측 오차를 제곱한 뒤 평균 내는 손실 함수. 큰 오차에 더 큰 벌점을 줌 | [01](01-loss-and-gradient-descent.md) | 손실 함수, 회귀 |
| 이진 교차엔트로피(BCE) | 0과 1을 구분하는 문제에서 정답 범주에 준 확률을 평가하는 손실 함수. 정답에 낮은 확률을 줄수록 손실이 커짐 | [06](06-keras-xor-training-comparison.md) | Sigmoid, 이진 분류 |
| 기울기(Gradient) | 파라미터를 조금 바꿀 때 손실이 어느 방향으로 얼마나 변하는지 나타내는 값 | [01](01-loss-and-gradient-descent.md) | 미분, 경사하강법 |
| 경사하강법(Gradient Descent) | 손실이 가장 빠르게 커지는 기울기의 반대 방향으로 파라미터를 반복 이동시키는 최적화 방법 | [01](01-loss-and-gradient-descent.md) | 기울기, 학습률 |
| 학습률(Learning Rate) | 한 번의 파라미터 갱신에서 얼마나 이동할지 정하는 크기. 너무 작으면 느리고 너무 크면 진동하거나 발산할 수 있음 | [01](01-loss-and-gradient-descent.md) | 경사하강법, 수렴 |
| 순전파(Forward Propagation) | 입력을 모델의 앞쪽부터 통과시켜 예측값과 손실을 계산하는 과정 | [02](02-forward-backprop-and-autodiff.md) | 역전파, 손실 함수 |
| 역전파(Backpropagation) | 최종 손실에서 출발해 각 파라미터가 손실에 미친 영향을 출력 쪽부터 거꾸로 계산하는 방법 | [02](02-forward-backprop-and-autodiff.md) | 연쇄법칙, 기울기 |
| 연쇄법칙(Chain Rule) | 여러 함수가 이어질 때 각 구간의 미분값을 곱해 전체 변화율을 구하는 미분 규칙 | [02](02-forward-backprop-and-autodiff.md) | 역전파, 자동미분 |
| 자동미분·`GradientTape` | 계산한 연산 경로를 기록하고 연쇄법칙을 적용해 지정한 변수에 대한 손실의 기울기를 자동으로 구하는 기능 | [02](02-forward-backprop-and-autodiff.md) | 역전파, `tf.Variable` |

## 텐서와 선형 계산

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 텐서(Tensor) | 스칼라·벡터·행렬을 포함하는 다차원 숫자 배열. TensorFlow 계산과 자동미분의 기본 재료 | [03](03-tensors-and-eager-execution.md) | rank, shape |
| rank | 텐서가 가진 축의 개수. 벡터는 rank 1, 행렬은 rank 2 | [03](03-tensors-and-eager-execution.md) | shape, axis |
| shape | 각 축에 값이 몇 개씩 있는지 나타내는 텐서의 모양. `(100, 2)`라면 샘플 100개와 특성 2개를 뜻할 수 있음 | [03](03-tensors-and-eager-execution.md) | rank, 배치 |
| 즉시 실행(Eager Execution) | 연산을 호출할 때 결과를 바로 계산해 중간값을 확인할 수 있는 실행 방식 | [03](03-tensors-and-eager-execution.md) | 디버깅, 계산 그래프 |
| `tf.constant` | 계산 중 내부 값을 바꾸지 않는 고정 텐서를 만드는 API | [04](04-tensor-apis-and-linear-transform.md) | `tf.Variable`, 텐서 |
| `tf.Variable`·`assign` | 학습 중 바뀌는 가중치처럼 갱신 가능한 텐서와, 그 변수의 내부 값을 교체하는 메서드 | [04](04-tensor-apis-and-linear-transform.md) | 가중치, 경사하강법 |
| `tf.linspace`·`tf.range` | `linspace`는 양 끝을 포함해 지정한 개수의 값을 만들고, `range`는 끝값 미만까지 지정한 간격으로 값을 만듦 | [04](04-tensor-apis-and-linear-transform.md) | 텐서 생성, 인덱스 |
| 행렬 곱(`tf.matmul`) | 왼쪽 행렬의 행과 오른쪽 행렬의 열을 내적해 새 행렬을 만드는 연산. 같은 위치끼리 곱하는 `*`와 다름 | [03](03-tensors-and-eager-execution.md) | $Wx+b$, shape |
| 브로드캐스팅(Broadcasting) | shape가 다른 작은 텐서를 계산 가능한 방향으로 확장해 함께 연산하는 규칙 | [04](04-tensor-apis-and-linear-transform.md) | 편향, shape |
| 선형 변환($Wx+b$) | 입력에 가중치 행렬을 곱해 조합하고 편향을 더해 한 층의 출력을 만드는 기본 계산 | [04](04-tensor-apis-and-linear-transform.md) | Dense, 행렬 곱 |

## 학습 반복과 데이터 파이프라인

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 에포크(Epoch) | 모델이 전체 학습 데이터를 처음부터 끝까지 한 번 사용한 횟수 | [05](05-epoch-batch-iteration-and-tfdata.md) | 배치, 이터레이션 |
| 배치(Batch) | 한 번의 학습 단계에서 모델이 함께 확인하는 데이터 묶음 | [05](05-epoch-batch-iteration-and-tfdata.md) | 배치 크기, 미니배치 |
| 이터레이션(Iteration) | 배치 하나로 순전파·역전파·파라미터 갱신을 수행한 한 번의 학습 단계 | [05](05-epoch-batch-iteration-and-tfdata.md) | 에포크, 가중치 갱신 |
| `tf.data.Dataset` | 입력과 레이블을 한 쌍으로 유지하며 변환·섞기·배치 만들기·미리 읽기를 연결하는 데이터 파이프라인 | [05](05-epoch-batch-iteration-and-tfdata.md) | `shuffle`, `prefetch` |
| `shuffle` | 학습 전에 샘플 순서를 섞는 데이터 변환. 입력과 레이블의 대응은 유지해야 함 | [05](05-epoch-batch-iteration-and-tfdata.md) | 섞기 버퍼, 배치 |
| `prefetch` | 모델이 현재 배치를 계산하는 동안 다음 배치를 미리 준비해 대기 시간을 줄이는 변환 | [05](05-epoch-batch-iteration-and-tfdata.md) | 데이터 파이프라인, 배치 |

## Keras 분류 모델과 평가

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| XOR | 두 입력이 서로 다를 때만 1이 되는 이진 분류 문제. 정답이 대각선으로 놓여 직선 하나로 나눌 수 없음 | [06](06-keras-xor-training-comparison.md) | 비선형성, ReLU |
| ReLU | 음수는 0으로, 양수는 그대로 통과시키는 활성화 함수. 은닉층에 꺾인 비선형 표현을 만듦 | [06](06-keras-xor-training-comparison.md) | XOR, 활성화 함수 |
| Adam | 기울기의 통계를 이용해 파라미터마다 갱신 보폭을 조절하는 최적화 방법 | [06](06-keras-xor-training-comparison.md) | SGD, 학습률 |
| Keras 학습 흐름 | `Sequential`로 층을 쌓고 `compile`로 학습 설정을 정한 뒤 `fit`·`evaluate`·`predict`로 학습·평가·예측하는 순서 | [06](06-keras-xor-training-comparison.md) | 손실 함수, 최적화 |
| Flatten | 28×28 같은 다차원 입력을 값의 순서를 유지한 채 1차원 특성 벡터로 펼치는 층. 학습 파라미터는 없음 | [07](07-mnist-dense-network.md) | MNIST, Dense |
| Dense | 이전 층의 모든 출력과 연결되어 $Wx+b$와 활성화 함수를 계산하는 완전연결층 | [07](07-mnist-dense-network.md) | 선형 변환, 파라미터 |
| Dropout | 훈련 중 일부 은닉 출력을 무작위로 0으로 만들어 특정 연결에 지나치게 의존하는 것을 줄이는 방법 | [07](07-mnist-dense-network.md) | 과적합, 훈련 모드 |
| Softmax | 여러 클래스의 점수를 합이 1인 확률로 바꾸는 출력 함수. 가장 큰 확률의 인덱스를 예측 클래스로 볼 수 있음 | [07](07-mnist-dense-network.md) | 다중 분류, 교차엔트로피 |
| 희소 범주형 교차엔트로피 | 정답이 원-핫 벡터가 아닌 정수 클래스일 때 정답 클래스에 준 확률을 평가하는 다중 분류 손실 함수 | [07](07-mnist-dense-network.md) | Softmax, MNIST |
