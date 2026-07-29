# 일반화와 딥러닝 성능 개선

## 한 줄 요약

검증 손실을 추적하고 조기 종료, 데이터 증강, 가중치 규제, 드롭아웃으로 과적합을 줄입니다.

## 학습과 추론

- 학습: 순전파, 손실 계산, 역전파, 옵티마이저 갱신을 반복합니다.
- 검증: 학습에 사용하지 않은 데이터로 과적합을 감시합니다.
- 추론: 학습된 파라미터로 순전파만 수행합니다.

GPU는 병렬 연산에 강하지만 데이터 전송과 작은 모델에서는 병목 때문에 항상 빠른 것은 아닙니다.

## 조기 종료

```python
from tensorflow.keras.callbacks import EarlyStopping

early_stopping = EarlyStopping(
    monitor="val_loss",
    patience=30,
    restore_best_weights=True,
)

model.fit(
    X_train, y_train,
    validation_data=(X_valid, y_valid),
    callbacks=[early_stopping],
    epochs=1000,
)
```

검증 손실이 더 좋아지지 않을 때 학습을 멈추고 가장 좋았던 가중치를 복원합니다.

## 데이터 증강

기존 데이터에 타당한 변형을 가해 다양성을 늘립니다. 표 형식 재료 데이터에서는 작은 노이즈, 재표본화, SMOTE 등을 고려할 수 있지만 물리적으로 불가능한 조합을 만들지 않도록 도메인 검증이 필요합니다.

## L2 가중치 규제

```python
from tensorflow.keras import regularizers

Dense(64, activation="relu", kernel_regularizer=regularizers.l2(1e-4))
```

큰 가중치에 벌점을 부여해 모델이 특정 피처에 과도하게 의존하는 것을 줄입니다.

## 드롭아웃

```python
tf.keras.layers.Dense(64, activation="relu"),
tf.keras.layers.Dropout(0.3),
```

학습 중 일부 뉴런을 확률적으로 끄고, 추론할 때는 모든 뉴런을 사용합니다.

## AutoML의 역할

AutoML은 모델 구조와 하이퍼파라미터 탐색을 자동화합니다. 빠른 기준 성능을 만드는 데 유용하지만 데이터 누수, 평가 설계, 도메인 타당성까지 자동으로 보장하지는 않습니다.

## 복습 질문 및 답변

**Q1. 조기 종료에서 `restore_best_weights=True`가 중요한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	학습이 멈춘 마지막 시점이 아니라 검증 손실이 가장 낮았던 시점의 모델을 복원하기 위해서입니다.
</details>

**Q2. 드롭아웃은 추론할 때도 뉴런을 끄나요?**

<details>
<summary>답</summary>
	일반적인 추론 모드에서는 드롭아웃을 비활성화하고 모든 뉴런을 사용합니다.
</details>

**Q3. 표 형식 재료 데이터 증강에 도메인 지식이 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	수치상 가능해 보이더라도 물리적으로 성립하지 않는 조성이나 물성을 만들면 모델이 잘못된 규칙을 학습할 수 있기 때문입니다.
</details>

## 다음 학습

[전체 프로젝트 파이프라인](08-end-to-end-cathode-classification-workflow.md)에서 모든 단계를 연결합니다.
