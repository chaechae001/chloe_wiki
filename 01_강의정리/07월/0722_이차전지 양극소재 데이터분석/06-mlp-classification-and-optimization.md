# MLP 분류 모델과 최적화

## 한 줄 요약

다층 퍼셉트론으로 비선형 결정 경계를 학습하고, 활성화 함수·학습률·옵티마이저가 성능에 미치는 영향을 이해합니다.

## 선형 모델에서 MLP로

선형 회귀는 비선형 관계를 충분히 표현하지 못합니다. MLP는 은닉층과 비선형 활성화 함수를 쌓아 복잡한 패턴을 학습합니다.

```python
import tensorflow as tf

model = tf.keras.Sequential([
    tf.keras.layers.Input(shape=(X_train.shape[1],)),
    tf.keras.layers.Dense(64, activation="relu"),
    tf.keras.layers.Dense(32, activation="relu"),
    tf.keras.layers.Dense(3, activation="softmax"),
])
```

## 원-핫 인코딩과 소프트맥스

```python
from tensorflow.keras.utils import to_categorical

y_train_onehot = to_categorical(y_train, num_classes=3)
```

출력층의 소프트맥스는 클래스별 점수를 합이 1인 확률로 변환합니다. 원-핫 라벨에는 `categorical_crossentropy`, 정수 라벨에는 `sparse_categorical_crossentropy`를 사용합니다.

## 학습 설정

```python
model.compile(
    optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
    loss="sparse_categorical_crossentropy",
    metrics=["accuracy"],
)

history = model.fit(
    X_train, y_train,
    epochs=150,
    batch_size=16,
    validation_data=(X_valid, y_valid),
)
```

## 활성화 함수

- Sigmoid: 출력이 0~1이지만 깊은 층에서 기울기 소실이 생기기 쉽습니다.
- Tanh: 출력 중심이 0이지만 포화 영역에서 기울기가 작아집니다.
- ReLU: 계산이 단순하고 은닉층의 기본 선택으로 널리 사용됩니다.
- Softmax: 다중 분류 출력층에서 클래스 확률을 만듭니다.

## 옵티마이저

- SGD: 기본적인 경사하강법, 학습률 설정이 중요합니다.
- AdaGrad: 자주 갱신된 파라미터의 학습률을 줄입니다.
- RMSprop: 최근 기울기를 반영해 AdaGrad의 과도한 감소를 완화합니다.
- Adam: Momentum과 RMSprop의 아이디어를 결합한 일반적인 기본 선택입니다.

## 복습 질문 및 답변

**Q1. 은닉층에 비선형 활성화 함수가 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	활성화 함수가 없으면 여러 선형층을 쌓아도 결국 하나의 선형 변환과 같아 복잡한 비선형 관계를 표현할 수 없기 때문입니다.
</details>

**Q2. 정수 라벨을 그대로 사용할 때 적합한 손실 함수는 무엇인가요?**

<details>
<summary>답</summary>
	다중 분류에서는 `sparse_categorical_crossentropy`를 사용하면 별도의 원-핫 인코딩 없이 정수 라벨을 사용할 수 있습니다.
</details>

**Q3. 학습률이 너무 크면 어떤 문제가 생기나요?**

<details>
<summary>답</summary>
	최적점을 지나치며 손실이 진동하거나 발산할 수 있습니다.
</details>

## 다음 학습

[일반화 성능 개선](07-generalization-and-deep-learning-improvements.md)에서 과적합을 줄이는 방법을 다룹니다.
