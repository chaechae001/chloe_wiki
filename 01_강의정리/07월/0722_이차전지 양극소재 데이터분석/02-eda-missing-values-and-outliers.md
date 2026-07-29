# 데이터 탐색, 결측치와 이상치 전처리

## 한 줄 요약

모델을 만들기 전에 자료형, 분포, 결측치, 이상치를 확인하고 학습 가능한 데이터로 정리합니다.

## 데이터 구조부터 확인하기

```python
df.shape
df.info()
df.describe(include="all")
df.nunique().sort_values()
```

`shape`는 규모, `info()`는 자료형과 결측치, `describe()`는 분포, `nunique()`는 범주 후보를 보여줍니다.

## 결측치 비율 계산

```python
missing_ratio = df.isna().mean().sort_values(ascending=False)
drop_cols = missing_ratio[missing_ratio > 0.30].index
df = df.drop(columns=drop_cols)
```

결측 비율이 높다고 무조건 제거하지는 않습니다. 중요한 도메인 변수라면 수집 방식과 결측 원인을 먼저 확인해야 합니다.

## 결측값 대체

```python
from sklearn.impute import IterativeImputer

numeric_cols = X_train.select_dtypes(include="number").columns
imputer = IterativeImputer(random_state=42)
X_train[numeric_cols] = imputer.fit_transform(X_train[numeric_cols])
X_test[numeric_cols] = imputer.transform(X_test[numeric_cols])
```

핵심은 `fit`을 학습 데이터에만 수행하는 것입니다. 전체 데이터에서 대체 기준을 학습하면 테스트 정보가 유출됩니다.

## Z-score로 이상치 찾기

```python
from scipy import stats
import numpy as np

z = np.abs(stats.zscore(X_train[numeric_cols], nan_policy="omit"))
keep = (z < 3.0).all(axis=1)
X_train_clean = X_train.loc[keep]
y_train_clean = y_train.loc[keep]
```

Z-score는 평균에서 표준편차 몇 배만큼 떨어져 있는지 나타냅니다. 분포가 비대칭이거나 표본이 작으면 IQR이나 도메인 기준이 더 안전할 수 있습니다.

## 클래스 균형 확인

```python
class_ratio = y.value_counts(normalize=True)
print(class_ratio)
```

정확도는 다수 클래스를 잘 맞히는 모델을 과대평가할 수 있습니다. 클래스별 재현율과 혼동행렬을 함께 확인합니다.

## 흔한 실수

- 숫자로 저장된 범주를 연속형 수치로 간주합니다.
- 테스트 데이터까지 포함해 결측치 대체 기준을 계산합니다.
- 이상치를 무조건 오류로 보고 제거합니다.
- 행을 제거한 뒤 `X`와 `y`의 인덱스를 맞추지 않습니다.

## 복습 질문 및 답변

**Q1. 결측 비율이 높은 열을 무조건 삭제하면 안 되는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	도메인상 중요한 정보일 수 있고, 결측 자체가 특정 현상을 나타낼 수도 있기 때문입니다.
</details>

**Q2. 결측값 대체기를 학습 데이터에만 `fit`해야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	테스트 데이터의 분포 정보가 전처리 기준에 섞이는 데이터 누수를 막기 위해서입니다.
</details>

**Q3. Z-score 3을 넘는 값을 모두 제거해도 되나요?**

<details>
<summary>답</summary>
	아닙니다. 분포 형태와 재료과학적 타당성을 확인한 뒤 오류인지 희귀하지만 유효한 관측치인지 판단해야 합니다.
</details>

## 다음 학습

[피처 엔지니어링](03-encoding-scaling-and-feature-engineering.md)으로 이어집니다.
