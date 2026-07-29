# 인코딩, 스케일링과 피처 엔지니어링

## 한 줄 요약

범주형 값을 수치로 바꾸고, 피처 스케일을 맞추며, 중복되거나 불필요한 열을 제거해 모델이 학습하기 좋은 표현을 만듭니다.

## 범주형 데이터 인코딩

순서가 없는 범주는 원-핫 인코딩이 기본입니다.

```python
encoded = pd.get_dummies(df, columns=["Spacegroup"], dtype=int)
```

`cat.codes`나 `LabelEncoder`는 모델이 숫자 크기에 순서가 있다고 오해할 수 있습니다. 트리 모델처럼 이 문제에 덜 민감한 경우도 있지만 의미를 먼저 따져야 합니다.

## 정규화와 표준화

| 방법 | 식 | 특징 |
|---|---|---|
| Min-Max 정규화 | `(x - min) / (max - min)` | 보통 0~1 범위, 이상치에 민감 |
| 표준화 | `(x - mean) / std` | 평균 0, 표준편차 1 |

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

KNN, 로지스틱 회귀, 신경망, PCA처럼 거리나 기울기에 의존하는 모델은 스케일의 영향을 크게 받습니다.

## 상관관계와 중복 피처

```python
corr = X_train.corr(numeric_only=True).abs()
upper = corr.where(np.triu(np.ones(corr.shape), k=1).astype(bool))
drop_cols = [col for col in upper.columns if (upper[col] > 0.75).any()]
```

상관관계가 높다고 자동으로 모두 삭제하지 않습니다. 목표와의 관계, 해석 가능성, 다중공선성 영향을 함께 봅니다.

## 식별자와 도메인 지식

- `Materials Id`: 식별자이므로 일반적으로 제거합니다.
- `Formula`: 그대로는 고유값이 많지만 원소 조성으로 분해하면 유용한 피처가 될 수 있습니다.
- `Spacegroup`: 결정 구조와 직접 관련되므로 목표 누수 여부를 확인해야 합니다.
- `Formation Energy`, `E Above Hull`: 안정성과 연관된 도메인 피처입니다.

## 전처리 파이프라인

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

preprocess = ColumnTransformer([
    ("num", StandardScaler(), numeric_cols),
    ("cat", OneHotEncoder(handle_unknown="ignore"), categorical_cols),
])
```

파이프라인은 학습·검증·테스트에 같은 변환을 적용하고 데이터 누수를 줄여줍니다.

## 복습 질문 및 답변

**Q1. 원-핫 인코딩이 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	순서가 없는 범주에 임의의 크기 관계를 부여하지 않고 각 범주의 존재 여부를 독립적으로 표현하기 위해서입니다.
</details>

**Q2. 스케일러는 왜 학습 데이터에만 `fit`하나요?**

<details>
<summary>답</summary>
	테스트 데이터의 평균과 표준편차가 학습 과정에 유출되는 것을 막기 위해서입니다.
</details>

**Q3. 화학식 열을 무조건 삭제하는 것이 최선인가요?**

<details>
<summary>답</summary>
	아닙니다. 문자열 자체는 부적절할 수 있지만 원소별 함량이나 조성 비율로 변환하면 강력한 도메인 피처가 될 수 있습니다.
</details>

## 다음 학습

[선형 회귀와 로지스틱 회귀](04-linear-and-logistic-models.md)에서 기본 모델을 만듭니다.
