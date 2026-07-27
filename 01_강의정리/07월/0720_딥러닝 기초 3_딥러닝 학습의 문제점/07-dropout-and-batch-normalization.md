# 드롭아웃과 배치 정규화 — 무작위 규제와 안정적인 학습을 구분하기

> 드롭아웃과 배치 정규화는 모두 신경망의 학습을 돕지만 같은 일을 하지는 않습니다. 하나는 학습 경로를 무작위로 바꾸고, 다른 하나는 층에 들어오는 값의 스케일을 조정합니다. 역할을 구분해야 조합도 근거 있게 선택할 수 있습니다.

`Dropout` · `Inverted Dropout` · `Batch Normalization` · `Training Mode` · `Inference Mode`

## 핵심요약

- Dropout은 학습할 때 일부 활성값을 무작위로 0으로 만들어 특정 뉴런 조합에 대한 과도한 의존을 줄이는 규제 기법이다.
- 현대적인 inverted dropout은 학습 중 남은 활성값을 $1/(1-p)$배 보정하므로 추론할 때 별도로 dropout 확률을 곱하지 않는다.
- 저장된 실행에서는 학습 모드로 같은 입력을 두 번 통과시킨 출력의 평균 절대 차이가 `0.4252278`, 추론 모드의 차이는 `0.0`이었다.
- Batch Normalization은 미니배치의 평균과 분산으로 중간값을 정규화하고, 학습 가능한 $gamma$와 $eta$로 표현력을 다시 조절한다.
- BatchNorm의 주된 역할은 학습 신호의 스케일을 안정시키는 것이며, Dropout처럼 직접 무작위 마스크를 적용하는 규제와는 목적과 동작이 다르다.
- 한 번의 저장 실험에서는 Dropout, BatchNorm, 결합 설정이 모두 L2보다 낮은 검증 정확도를 보였다. 여러 기법을 조합하면 항상 우월하다고 단정할 수 없다.

## 1. Dropout은 학습할 때만 경로를 무작위로 바꾼다

### 1) 정의

Dropout은 학습 단계에서 은닉층 활성값 일부를 확률적으로 0으로 만드는 기법입니다. 매번 다른 유닛이 꺼지므로 모델은 특정 경로 하나에만 의존하기 어려워지고, 여러 부분 네트워크를 번갈아 학습하는 것과 비슷한 효과를 얻습니다.

Dropout 비율을 $p$라고 하고 각 활성값을 유지할지 나타내는 마스크를 $m$이라고 하면 다음처럼 표현할 수 있습니다.

$$
m_i \sim \operatorname{Bernoulli}(1-p)
$$

$$
h_i^{\text{train}} = \frac{m_i h_i}{1-p}
$$

$m_i=0$이면 해당 활성값은 꺼지고, $m_i=1$이면 살아남습니다. 분모의 $1-p$는 남은 활성값의 크기를 보정합니다.

### 2) 왜 필요한가

규제가 없는 큰 신경망은 훈련 데이터의 잡음까지 외울 수 있습니다. 또한 여러 뉴런이 언제나 함께 작동하면 서로의 존재를 전제로 한 공동 적응이 생길 수 있습니다.

Dropout은 학습할 때 연결 조합을 계속 바꿉니다. 각 뉴런은 특정 동료가 항상 켜져 있다고 기대할 수 없으므로, 더 다양한 경로에서 유용한 표현을 학습하도록 압력을 받습니다.

### 3) 핵심 흐름 재구성

Dropout의 동작은 학습 모드와 추론 모드로 나누어 이해해야 합니다.

| 모드 | 마스크 적용 | 출력 보정 | 같은 입력을 반복한 결과 |
|---|---|---|---|
| 학습 `training=True` | 일부 활성값을 무작위로 0으로 만듦 | 살아남은 값을 $1/(1-p)$배 보정 | 일반적으로 달라질 수 있음 |
| 추론 `training=False` | 모든 활성값을 사용 | 별도 확률 곱셈 없음 | 다른 확률적 연산이 없다면 같음 |

이 방식을 **inverted dropout**이라고 합니다. 기대값을 계산하면 학습 중에도 평균적인 활성값 크기가 유지됩니다.

$$
\mathbb{E}\left[\frac{m_i h_i}{1-p}\right]
= \frac{(1-p)h_i}{1-p}
= h_i
$$

따라서 추론 단계에서 출력에 $1-p$나 $p$를 다시 곱하면 오히려 스케일을 두 번 조정하게 됩니다. 현재 널리 쓰이는 Keras의 Dropout은 이 보정을 학습 단계에서 처리합니다.

### 4) 쉬운 예시

발표 준비를 다섯 명이 늘 같은 역할로 한다고 생각해 봅시다. 한 명만 자료 구조를 이해한다면 그 사람이 빠졌을 때 팀 전체가 흔들립니다. 연습할 때마다 임의의 구성원이 쉬고 남은 사람이 역할을 나누면, 여러 사람이 핵심 업무를 익히게 됩니다.

실전 발표에서는 모든 구성원이 참여합니다. 연습 때 인원이 줄었다는 이유로 실전 발표자의 목소리를 다시 일정 비율로 낮추지는 않습니다. 학습 중 이미 보정했다는 점이 inverted dropout의 핵심입니다.

### 5) 코드 예시

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

tf.random.set_seed(42)

model = keras.Sequential([
    layers.Input(shape=(4,)),
    layers.Dense(8, activation="relu"),
    layers.Dropout(0.5),
    layers.Dense(1, activation="sigmoid"),
])

x = tf.ones((5, 4))

train_a = model(x, training=True)
train_b = model(x, training=True)
infer_a = model(x, training=False)
infer_b = model(x, training=False)

train_diff = tf.reduce_mean(tf.abs(train_a - train_b))
infer_diff = tf.reduce_mean(tf.abs(infer_a - infer_b))

print("training difference:", float(train_diff))
print("inference difference:", float(infer_diff))
```

학습 모드에서는 두 호출의 마스크가 달라 출력 차이가 0보다 클 수 있습니다. 추론 모드에서는 Dropout이 꺼지므로 다른 확률적 요소가 없다면 차이가 0이 됩니다. 학습 모드의 정확한 숫자는 초기 가중치와 난수 상태에 따라 달라집니다.

### 6) 헷갈리는 점

Dropout 비율 `0.5`는 출력을 절반으로 만드는 값이 아니라 각 활성값을 끌 확률입니다. 학습 시 살아남은 활성값은 자동으로 확대되며, 추론에서는 모든 활성값을 그대로 사용합니다.

Dropout도 무조건 많이 적용하면 좋은 것이 아닙니다. 비율이 지나치게 높으면 학습에 사용할 신호가 부족해져 훈련 성능과 검증 성능이 함께 낮아질 수 있습니다.

### 7) 한 줄 정리

> Dropout은 학습 중 무작위 마스크와 자동 스케일 보정으로 경로 의존을 낮추고, 추론 중에는 모든 활성값을 결정론적으로 사용한다.

## 2. Batch Normalization은 층별 신호의 스케일을 조절한다

### 1) 정의

Batch Normalization은 미니배치 안의 중간 활성값을 평균 0, 분산 1에 가까운 값으로 정규화한 뒤, 학습 가능한 스케일과 이동값을 적용하는 층입니다.

미니배치 $B$의 평균과 분산은 다음과 같습니다.

$$
\mu_B = \frac{1}{|B|}\sum_{i \in B} x_i
$$

$$
\sigma_B^2 = \frac{1}{|B|}\sum_{i \in B}(x_i-\mu_B)^2
$$

정규화와 재조정은 다음 두 단계로 이어집니다.

$$
\hat{x}_i = \frac{x_i-\mu_B}{\sqrt{\sigma_B^2+\varepsilon}}
$$

$$
y_i = \gamma\hat{x}_i+\beta
$$

$\varepsilon$은 0으로 나누는 문제를 막는 작은 상수입니다. $\gamma$는 크기를 조절하고 $\beta$는 위치를 이동합니다.

### 2) 왜 필요한가

깊은 네트워크에서는 앞쪽 층의 가중치가 바뀔 때 뒤쪽 층이 받는 값의 분포도 계속 달라집니다. 값이 지나치게 커지거나 작아지면 활성화 함수와 기울기가 불안정해질 수 있습니다.

BatchNorm은 층 사이 신호의 크기를 일정한 범위로 맞춰, 학습률과 초기화에 대한 민감도를 줄이고 최적화를 안정시키는 데 도움을 줄 수 있습니다. 다만 모든 모델에서 속도나 일반화 성능을 반드시 개선하는 것은 아닙니다.

### 3) 핵심 흐름 재구성

학습 모드에서는 현재 미니배치의 평균과 분산을 사용합니다. 동시에 추론에 사용할 이동 평균과 이동 분산을 갱신합니다.

추론 모드에서는 입력 한 건의 통계나 현재 배치 통계를 다시 계산하지 않고, 학습 중 누적한 이동 통계를 사용합니다. 따라서 BatchNorm도 `training` 상태에 따라 계산 경로가 달라집니다.

여기서 $\gamma$와 $\beta$는 옵티마이저가 역전파로 조정하는 **학습 파라미터**입니다. 반면 `momentum`과 $\varepsilon$ 같은 설정은 사용자가 선택하는 **하이퍼파라미터**입니다.

| 요소 | 역할 | 학습 여부 |
|---|---|---|
| $\mu_B$, $\sigma_B^2$ | 현재 학습 배치의 통계 | 직접 학습하지 않음 |
| 이동 평균·이동 분산 | 추론 시 사용할 누적 통계 | 경사 하강이 아닌 이동 갱신 |
| $\gamma$, $\beta$ | 정규화된 값을 확대·축소하고 이동 | 역전파로 학습 |
| $\varepsilon$, `momentum` | 수치 안정성과 이동 통계 갱신 정도 설정 | 사용자가 정하는 하이퍼파라미터 |

### 4) 쉬운 예시

서로 다른 반의 시험 점수를 비교한다고 생각해 봅시다. 한 반은 평균이 90점이고 다른 반은 평균이 60점이면 원점수만으로 상대적 위치를 비교하기 어렵습니다. 각 반의 평균과 퍼짐을 기준으로 바꾸면 학생이 반 안에서 어느 위치인지 볼 수 있습니다.

그 뒤 $\gamma$와 $\beta$는 모든 반을 무조건 평균 0, 분산 1에 묶어 두지 않고, 다음 계산에 유리한 크기와 위치를 모델이 다시 선택하게 합니다.

### 5) 코드 예시

```python
from tensorflow import keras
from tensorflow.keras import layers

model = keras.Sequential([
    layers.Input(shape=(30,)),
    layers.Dense(64, use_bias=False),
    layers.BatchNormalization(),
    layers.Activation("relu"),
    layers.Dense(1, activation="sigmoid"),
])

model.compile(
    optimizer="adam",
    loss="binary_crossentropy",
    metrics=["accuracy"],
)
```

이 구조에서는 `Dense → BatchNormalization → ReLU` 순서로 값을 전달합니다. BatchNorm 뒤에는 학습 가능한 이동값 $\beta$가 있으므로 앞 Dense 층의 편향을 생략할 수 있습니다.

### 6) 헷갈리는 점

BatchNorm이 입력 데이터 전처리를 대신하는 것은 아닙니다. 입력 정규화는 데이터 전체의 특성 단위를 맞추는 작업이고, BatchNorm은 네트워크 내부의 중간값을 배치 단위로 처리하는 층입니다.

작은 배치에서는 평균과 분산 추정이 흔들릴 수 있습니다. 학습 데이터와 운영 데이터의 분포가 크게 달라지면 저장된 이동 통계가 실제 입력을 잘 나타내지 못할 수도 있습니다. 시계열처럼 배치 구성 자체가 의미를 가지거나 배치 크기가 매우 작은 환경에서는 다른 정규화 방법도 검토해야 합니다.

### 7) 한 줄 정리

> BatchNorm은 배치 통계로 중간 신호를 정규화하고 학습 가능한 $\gamma$·$\beta$로 다시 조정해 최적화의 안정성을 돕는다.

## 3. Dropout과 BatchNorm의 역할·한계·조합 판단

### 1) 정의

두 기법은 모두 학습 결과에 영향을 주지만 개입 지점이 다릅니다. Dropout은 활성값에 확률적 마스크를 적용하는 명시적 규제이고, BatchNorm은 활성값의 분포를 정돈하는 정규화 층입니다.

### 2) 왜 필요한가

기법의 이름만 보고 함께 넣으면 원인을 분석하기 어려워집니다. 과적합이 문제인지, 신호 스케일과 최적화가 문제인지 먼저 구분해야 합니다. 그래야 검증 실험에서 바꿀 변수를 줄이고 결과를 설명할 수 있습니다.

### 3) 핵심 흐름 재구성

| 관점 | Dropout | Batch Normalization |
|---|---|---|
| 주된 목적 | 공동 적응 완화와 과적합 억제 | 중간 신호 안정화와 최적화 보조 |
| 학습 중 동작 | 무작위 마스크 적용 | 현재 배치 평균·분산 사용 |
| 추론 중 동작 | 마스크 없이 전체 활성값 사용 | 누적한 이동 통계 사용 |
| 학습 파라미터 | 없음 | $\gamma$, $\beta$ |
| 대표 하이퍼파라미터 | dropout 비율 $p$ | `momentum`, $\varepsilon$ |
| 주요 한계 | 높은 비율에서 정보 손실·학습 지연 | 작은 배치와 분포 변화에 민감할 수 있음 |

저장된 비교 실행은 훈련 샘플 120개, 검증 샘플 400개, 특성 30개의 작은 이진 분류 조건에서 진행되었습니다.

| 설정 | 훈련 정확도 | 검증 정확도 | 격차 |
|---|---:|---:|---:|
| 규제 없음 | 1.000 | 0.770 | 0.230 |
| Dropout | 1.000 | 0.810 | 0.190 |
| BatchNorm | 0.992 | 0.752 | 0.239 |
| BatchNorm+Dropout+L2 | 1.000 | 0.788 | 0.212 |
| L2 | 1.000 | 0.842 | 0.158 |

이 한 번의 실행에서는 Dropout이 규제 없는 모델보다 검증 정확도가 4.0%p 높았지만, BatchNorm 단독은 1.8%p 낮았습니다. 결합 설정은 기준 모델보다 1.8%p 높았지만 L2의 0.842에는 미치지 못했습니다.

결합 설정에는 BatchNorm, Dropout, L2가 동시에 들어갔으므로 결과 차이를 어느 하나의 효과로 분리할 수 없습니다. 난수 초기값, 규제 강도, 배치 크기, 학습률을 바꾼 반복 실험 없이 “조합이 가장 좋다”라고 결론 내릴 수 없습니다.

### 4) 쉬운 예시

Dropout은 연습할 때 팀 구성원을 무작위로 바꾸는 방법이고, BatchNorm은 팀마다 다른 점수 기준을 일정하게 맞추는 방법과 비슷합니다. 두 방법을 함께 쓴다고 항상 최고 점수가 되는 것은 아닙니다. 구성원 교체가 너무 잦으면 오히려 익숙해질 시간이 부족하고, 기준 보정도 작은 팀에서는 불안정할 수 있습니다.

### 5) 코드 예시

```python
from tensorflow import keras
from tensorflow.keras import layers, regularizers

def build_classifier(dropout_rate=0.0, use_batchnorm=False, l2_rate=0.0):
    blocks = [layers.Input(shape=(30,))]

    for _ in range(3):
        blocks.append(layers.Dense(
            64,
            use_bias=not use_batchnorm,
            kernel_regularizer=regularizers.l2(l2_rate) if l2_rate else None,
        ))
        if use_batchnorm:
            blocks.append(layers.BatchNormalization())
        blocks.append(layers.Activation("relu"))
        if dropout_rate:
            blocks.append(layers.Dropout(dropout_rate))

    blocks.append(layers.Dense(1, activation="sigmoid"))
    return keras.Sequential(blocks)
```

비교할 때는 한 번에 하나의 설정만 바꾸는 실험부터 시작합니다. 예를 들어 기준 모델과 Dropout 모델을 먼저 비교하고, 그다음 BatchNorm을 독립적으로 비교해야 각 변화의 영향을 해석하기 쉽습니다.

### 6) 헷갈리는 점

BatchNorm에서 나타날 수 있는 규제 효과는 배치 통계의 변동에서 부수적으로 생깁니다. 이를 Dropout과 같은 목적과 강도의 규제로 간주하면 안 됩니다. 반대로 Dropout은 신호의 평균과 분산을 정규화하지 않습니다.

검증 정확도 하나만으로도 충분하지 않습니다. 검증 손실 곡선, 여러 난수 시드의 평균과 변동, 클래스별 지표, 추론 지연, 운영 배치 특성을 함께 확인해야 합니다.

### 7) 한 줄 정리

> Dropout은 확률적 규제, BatchNorm은 신호 안정화가 중심이며, 조합의 가치는 같은 조건의 반복 검증으로 판단해야 한다.

## 코드로 보기 — 학습·추론 모드와 검증 성능 함께 확인하기

```python
import tensorflow as tf

sample = x_valid[:5]

train_out_1 = model(sample, training=True)
train_out_2 = model(sample, training=True)
infer_out_1 = model(sample, training=False)
infer_out_2 = model(sample, training=False)

train_difference = tf.reduce_mean(tf.abs(train_out_1 - train_out_2))
infer_difference = tf.reduce_mean(tf.abs(infer_out_1 - infer_out_2))

print("training difference:", float(train_difference))
print("inference difference:", float(infer_difference))

valid_loss, valid_accuracy = model.evaluate(x_valid, y_valid, verbose=0)
print("validation accuracy:", valid_accuracy)
```

### 코드 목적

같은 입력을 학습 모드와 추론 모드로 각각 두 번 전달해 Dropout의 확률적 동작을 확인하고, 별도의 검증 데이터에서 일반화 성능을 함께 확인합니다.

### 코드 흐름

1. 같은 검증 입력 다섯 건을 준비한다.
2. `training=True`로 두 번 호출해 Dropout이 켜진 출력을 비교한다.
3. `training=False`로 두 번 호출해 추론 출력을 비교한다.
4. 평균 절대 차이와 검증 정확도를 함께 기록한다.
5. 여러 난수 시드와 설정에서 같은 절차를 반복한다.

### 실행 결과 해석

저장된 실행에서는 학습 모드 출력의 평균 절대 차이가 `0.4252278`, 추론 모드의 차이가 `0.0`이었습니다. 전자는 호출마다 다른 Dropout 마스크가 적용되었다는 관찰이고, 후자는 추론에서 Dropout이 비활성화되어 동일 계산이 수행되었다는 관찰입니다.

`0.4252278` 자체가 좋은 모델의 기준은 아닙니다. 값의 크기는 가중치, 입력, Dropout 비율과 모델 구조에 따라 달라집니다. 이 결과는 학습·추론 모드의 동작 차이를 확인하는 기능 검사로 해석해야 합니다.

또한 추론 모드 차이가 0이라는 사실은 정확도나 일반화 성능을 보장하지 않습니다. 저장된 비교에서 Dropout 검증 정확도는 0.810, BatchNorm은 0.752, 결합 설정은 0.788이었으며, 그 실행에서는 L2가 0.842로 가장 높았습니다.

### 실무 연결

모델을 서비스에 배포할 때는 평가와 예측 경로가 추론 모드인지 확인해야 합니다. Dropout이 실수로 켜져 있으면 같은 요청의 예측 확률이 달라질 수 있고, BatchNorm이 학습 모드로 동작하면 요청 배치의 구성에 따라 결과가 흔들릴 수 있습니다.

모델 실험에서는 `기준 모델 → Dropout만 추가 → BatchNorm만 추가 → 필요한 조합` 순으로 비교하고, 각 설정을 여러 시드로 반복합니다. 운영 배치 크기가 학습 때와 다르거나 입력 분포가 변한다면 BatchNorm의 이동 통계와 성능 저하도 점검합니다.

## 직접 해보기

1. Dropout 비율이 0.25일 때 학습 중 활성값 하나가 살아남을 확률과, 살아남은 값에 적용되는 inverted dropout 배율을 계산하세요.
2. 같은 입력을 학습 모드로 두 번 전달했을 때 평균 차이가 `0.4252278`, 추론 모드에서는 `0.0`이었다면 각각 무엇을 의미하나요?
3. BatchNorm+Dropout+L2 모델이 단독 기법보다 높은 성능을 보였을 때도 즉시 “세 기법의 조합이 우월하다”고 결론 내리면 안 되는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 살아남을 확률은 $1-0.25=0.75$입니다. 살아남은 활성값에는 $1/0.75\approx1.333$배가 적용되어 학습 중 기대 스케일을 유지합니다.
2. 학습 모드의 양수 차이는 호출마다 다른 Dropout 마스크가 적용되었음을 보여 줍니다. 추론 모드의 0은 Dropout이 꺼져 동일한 결정론적 계산을 했음을 보여 줍니다. 두 값만으로 모델 성능의 좋고 나쁨을 판단할 수는 없습니다.
3. 세 설정을 동시에 바꾸면 성능 변화의 원인을 분리할 수 없습니다. 같은 데이터 분할에서 각 기법을 하나씩 추가하는 절제 실험과 여러 시드 반복, 별도 테스트 평가가 필요합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 일반 Dropout 설명 vs inverted dropout | 추론 때 별도 축소를 설명하는 오래된 관점 vs 학습 때 살아남은 값을 확대해 추론 스케일을 그대로 쓰는 현대 구현 |
| 학습 모드 vs 추론 모드 | Dropout 마스크·배치 통계를 사용 vs Dropout 비활성화·BatchNorm 이동 통계를 사용 |
| Dropout vs BatchNorm | 확률적 마스크로 공동 적응을 낮춤 vs 배치 통계로 중간 신호를 정규화 |
| $\gamma,\beta$ vs `momentum`, $\varepsilon$ | 역전파로 정해지는 학습 파라미터 vs 사용자가 설정하는 하이퍼파라미터 |
| 배치 정규화 vs 입력 정규화 | 네트워크 중간 활성값을 배치 단위로 처리 vs 데이터 특성을 학습 전에 공통 기준으로 변환 |
| 결정론적 출력 vs 좋은 일반화 | 같은 입력에 같은 출력 제공 vs 보지 않은 데이터에서도 높은 성능 유지 |
| 기법 조합 vs 절제 실험 | 여러 변수를 한꺼번에 변경 vs 한 번에 하나씩 바꿔 효과를 분리 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [과적합과 L1·L2 정규화](06-overfitting-and-regularization.md)
- 신호 스케일의 출발점: [가중치 초기화와 신호 전파](05-weight-initialization-and-signal-propagation.md)
- 배치와 통계의 연결: [미니배치와 최적화 기초](02-mini-batch-and-optimization-basics.md)
- 전체 진단 흐름: [딥러닝 학습 실패 모드](01-deep-learning-training-failure-modes.md)
- 함께 보면 좋은 키워드: `일반화`, `이동 평균`, `절제 실험`

## 셀프 체크

- [ ] Dropout이 학습과 추론에서 어떻게 다르게 동작하는지 설명할 수 있다.
- [ ] inverted dropout의 $1/(1-p)$ 보정 이유를 기대값으로 설명할 수 있다.
- [ ] BatchNorm의 정규화 수식에서 평균, 분산, $\varepsilon$의 역할을 말할 수 있다.
- [ ] $\gamma$와 $\beta$가 학습 파라미터임을 설명할 수 있다.
- [ ] BatchNorm이 추론할 때 이동 통계를 사용하는 이유를 말할 수 있다.
- [ ] 저장된 실행 결과를 성능 순위가 아닌 조건부 관찰로 해석할 수 있다.
- [ ] Dropout과 BatchNorm을 함께 쓴다고 항상 좋아지는 것은 아님을 설명할 수 있다.
- [ ] 배포 시 모델의 학습·추론 모드를 점검할 수 있다.

### 복습 질문 및 답변

**Q1. inverted dropout에서는 왜 추론할 때 출력에 유지 확률을 다시 곱하지 않나요?**

<details>
<summary>답</summary>

학습할 때 살아남은 활성값을 이미 $1/(1-p)$배 확대해 원래 활성값과 기대 스케일을 맞췄기 때문입니다. 추론에서는 모든 활성값을 그대로 사용하며, 유지 확률을 다시 곱하면 이중 보정이 됩니다.

</details>

**Q2. BatchNorm의 $\gamma$와 $\beta$는 왜 필요한가요?**

<details>
<summary>답</summary>

정규화된 값을 항상 평균 0, 분산 1 부근에만 묶어 두지 않고 다음 층에 유리한 크기와 위치로 다시 바꾸기 위해 필요합니다. 두 값은 손실을 줄이는 방향으로 역전파를 통해 학습됩니다.

</details>

**Q3. 저장된 실행에서 L2의 검증 정확도가 0.842로 가장 높았다는 결과를 어떻게 활용해야 하나요?**

<details>
<summary>답</summary>

해당 데이터 분할과 하이퍼파라미터에서 L2가 유망했다는 후보 근거로 활용합니다. 이를 보편적 순위로 일반화하지 않고 여러 난수 시드, 규제 강도, 배치 크기로 반복한 뒤 모델 선택에 쓰지 않은 별도 테스트 세트에서 최종 확인해야 합니다.

</details>

## 한 줄 정리

> Dropout은 학습 중 경로를 무작위로 바꾸고 inverted scaling으로 기대값을 유지하며, BatchNorm은 배치 통계와 학습 가능한 $\gamma$·$\beta$로 신호를 안정시킨다. 두 기법의 조합은 역할을 구분한 통제 실험으로 검증해야 한다.
