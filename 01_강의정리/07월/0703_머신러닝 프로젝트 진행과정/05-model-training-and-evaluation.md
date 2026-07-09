# 모델 학습과 평가

> 데이터를 나누고, 모델을 학습시키고, 성능을 제대로 읽는 것까지가 프로젝트의 마지막 단추다.

`train_test_split` · `교차검증` · `R²` · `정밀도·재현율·F1` · `Pipeline`

## 핵심요약

- 전체 데이터로 학습하고 같은 데이터로 평가하면 과적합을 걸러낼 수 없다.
- `train_test_split`으로 훈련/테스트를 나누고, 보통 7:3 또는 8:2 비율을 쓴다.
- K-fold 교차검증은 데이터를 여러 번 나눠 평가해 성능을 더 안정적으로 추정한다.
- 회귀는 R²·RMSE로, 분류는 정확도·정밀도·재현율·F1로 평가한다.
- 정확도만 보면 불균형 데이터에서 성능을 오해할 수 있다.

## 1. 데이터 분리(train_test_split)

### 1) 정의

**데이터 분리**는 전체 데이터를 학습에 쓸 **훈련 세트**와 성능 확인에 쓸 **테스트 세트**로 나누는 과정입니다.

### 2) 왜 필요한가

전체 데이터로 학습한 뒤 같은 데이터로 평가하면, 모델이 답을 외운 상태에서 시험을 보는 것과 같습니다. 이러면 과적합(훈련 데이터에만 잘 맞는 상태)을 걸러낼 수 없습니다. 처음 보는 테스트 데이터로 평가해야 진짜 일반화 성능을 알 수 있습니다.

### 3) 코드 예시

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.3,        # 테스트 세트 비율 30%
    random_state=42       # 재현성을 위한 시드 고정
)
```

| 파라미터 | 설명 |
|---------|------|
| `*arrays` | 분리할 데이터 (특성 X, 정답 y) |
| `test_size` | 테스트 세트가 차지할 비율 |
| `random_state` | 매번 같은 분할을 얻기 위한 난수 시드 |

분류 문제에서는 `stratify=y`를 추가하면 훈련/테스트에서 정답 클래스 비율을 똑같이 유지할 수 있습니다. 불균형 데이터에서 특히 유용합니다.

### 4) 한 줄 정리

> 데이터 분리는 "처음 보는 데이터"를 남겨 두어 모델의 진짜 실력을 재는 장치다.

## 2. 교차검증(K-fold Cross Validation)

### 1) 정의

**K-fold 교차검증**은 데이터를 K개 조각(fold)으로 나눈 뒤, 한 조각씩 돌아가며 검증에 쓰고 나머지로 학습하는 과정을 K번 반복하는 방법입니다.

### 2) 왜 필요한가

단 한 번의 분리로 평가하면, 어쩌다 쉬운 데이터가 테스트에 몰려 운 좋게 좋은 점수가 나올 수 있습니다. 교차검증은 여러 번 나눠 평균을 내므로 성능을 더 안정적으로 추정하고 과적합을 완화합니다.

### 3) 핵심 흐름 재구성

5-fold라면 데이터를 5조각으로 나눠, 각 회차마다 4조각으로 학습하고 남은 1조각으로 검증합니다. 이렇게 5번 하면 5개의 점수가 나오고, 그 평균과 편차로 성능을 판단합니다. 편차가 작을수록 안정적인 모델입니다.

### 4) 코드 예시

```python
from sklearn.model_selection import KFold, cross_val_score
from sklearn.linear_model import LinearRegression

kf = KFold(n_splits=5, shuffle=True, random_state=42)

scores = cross_val_score(LinearRegression(), X, y, cv=kf, scoring='r2')
print(scores.mean(), scores.std())   # 평균 성능과 편차
```

### 5) 한 줄 정리

> 교차검증은 데이터를 여러 번 나눠 평가해, 한 번의 운에 흔들리지 않는 성능 추정을 준다.

## 3. 회귀 평가 지표

### 1) 정의

- **R²(결정계수)**: 모델이 데이터의 변동을 얼마나 설명하는지. 1에 가까울수록 좋습니다.
- **RMSE(평균제곱근오차)**: 예측이 실제와 평균적으로 얼마나 벗어났는지. 낮을수록 좋습니다.

### 2) 코드 예시

```python
from sklearn.metrics import r2_score, mean_squared_error
import numpy as np

r2 = r2_score(y_test, y_pred)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
```

### 3) 실행 결과 해석

R²가 0.7이면 모델이 정답의 변동 중 약 70%를 설명한다는 뜻입니다. RMSE가 0.9달러면 예측 팁이 평균적으로 실제와 0.9달러쯤 차이 난다는 의미입니다. 두 지표를 함께 봐야 "얼마나 설명하는지"와 "얼마나 틀리는지"를 모두 알 수 있습니다.

### 4) 한 줄 정리

> 회귀는 R²로 설명력을, RMSE로 오차 크기를 함께 읽는다.

## 4. 분류 평가 지표

### 1) 정의

분류에서는 정확도 하나로 부족합니다. 다음 네 지표를 함께 봅니다.

| 지표 | 의미 |
|------|------|
| 정확도(Accuracy) | 전체 중 맞춘 비율 |
| 정밀도(Precision) | "생존이라 예측한 것" 중 실제 생존 비율 |
| 재현율(Recall) | "실제 생존" 중 모델이 생존이라 맞춘 비율 |
| F1-score | 정밀도와 재현율의 조화평균 (균형 지표) |

### 2) 왜 필요한가

정답 클래스가 불균형하면 정확도가 성능을 크게 오해하게 만듭니다. 예를 들어 불량이 1%인 데이터에서 모두 "정상"이라 찍으면 정확도는 99%지만 불량은 하나도 못 잡습니다. 이때 재현율이 실체를 드러냅니다.

### 3) 핵심 흐름 재구성

**혼동 행렬(Confusion Matrix)**은 실제값과 예측값을 교차한 표로, 어디서 틀렸는지를 보여줍니다. 예측이 실제 정답과 얼마나 겹치는지를 칸별로 확인하면 정밀도와 재현율의 의미가 눈에 들어옵니다.

### 4) 쉬운 예시

병을 진단하는 모델을 생각해 봅시다. 정밀도가 낮으면 "병이라 진단했는데 실제로는 건강한" 오진이 많다는 뜻이고, 재현율이 낮으면 "실제 환자를 놓친" 경우가 많다는 뜻입니다. 의료에서는 놓치면 위험하므로 재현율이 특히 중요합니다.

### 5) 코드 예시

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

acc = accuracy_score(y_test, y_pred)
precision = precision_score(y_test, y_pred)
recall = recall_score(y_test, y_pred)
f1 = f1_score(y_test, y_pred)
```

### 6) 한 줄 정리

> 분류는 정확도만 믿지 말고 정밀도·재현율·F1을 함께 보아야 진짜 성능을 안다.

## 코드로 보기 — 전처리와 모델을 하나로 묶는 Pipeline

앞선 글에서 결측치 처리, 스케일링, 더미 변환을 배웠습니다. 실무에서는 이 과정을 **Pipeline**으로 묶어 훈련/테스트에 일관되게 적용합니다. 특히 결측치를 채울 때 훈련·테스트의 평균이 서로 다르면 문제가 되는데, Pipeline이 이를 자동으로 올바르게 처리합니다.

```python
from sklearn.model_selection import train_test_split
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.tree import DecisionTreeClassifier

numeric_features = ['pclass', 'fare', 'age']   # 결측 가능한 수치형
categorical_features = ['embarked']             # 결측 가능한 범주형

# 수치형: 중앙값으로 결측 채우고 표준화
numeric_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

# 범주형: 최빈값으로 결측 채우고 0/1 인코딩
categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

# 컬럼별로 다른 전처리를 지정
preprocessor = ColumnTransformer([
    ('num', numeric_transformer, numeric_features),
    ('cat', categorical_transformer, categorical_features)
])

# 전처리 + 모델을 하나의 흐름으로 결합
pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('model', DecisionTreeClassifier(random_state=42))
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)
```

### 코드 목적

수치형과 범주형에 서로 다른 전처리를 적용하고, 그 뒤 분류 모델까지 하나의 객체로 묶어 데이터 누수 없이 학습·예측하려는 코드입니다.

### 코드 흐름

1. 수치형·범주형 변수를 나눈다.
2. 각각에 맞는 결측치 처리와 인코딩/스케일링을 정의한다.
3. `ColumnTransformer`로 컬럼별 전처리를 묶는다.
4. 전처리와 모델을 `Pipeline`으로 결합해 한 번에 학습·예측한다.

### 실행 결과 해석

`pipeline.fit()`을 호출하면 훈련 데이터의 중앙값·최빈값·스케일 기준이 학습되고, `predict()` 시 테스트 데이터에는 그 기준이 그대로 적용됩니다. 덕분에 훈련/테스트의 결측 대체값이 어긋나 생기는 데이터 누수를 자동으로 막습니다.

### 실무 연결

실제 서비스에서는 새 데이터가 계속 들어오므로, 전처리와 모델을 하나로 묶어 두면 배포와 재현이 쉽습니다. Pipeline은 코드가 길어질 때 실수를 줄이는 표준 도구입니다.

## 직접 해보기

1. 전체 데이터로 학습하고 같은 데이터로 평가하면 어떤 문제가 생기나요?
2. 5-fold 교차검증에서 나온 5개 점수의 편차가 크다면 무엇을 의미하나요?
3. 불량 1%, 정상 99%인 데이터에서 모델이 모두 "정상"이라 예측했습니다. 정확도와 재현율은 각각 대략 얼마이며, 어떤 지표를 봐야 하나요?

<details>
<summary>정답 보기</summary>

1. 모델이 답을 외운 상태로 시험을 보는 셈이라 과적합을 걸러낼 수 없습니다. 처음 보는 테스트 데이터로 평가해야 일반화 성능을 알 수 있습니다.
2. 데이터를 어떻게 나누느냐에 따라 성능이 크게 흔들린다는 뜻이며, 모델이 불안정하거나 데이터가 작을 수 있습니다.
3. 정확도는 약 99%(정상을 다 맞힘)이지만, 불량에 대한 재현율은 0%입니다. 정확도가 성능을 오해하게 하므로 재현율과 F1을 함께 봐야 합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 훈련 세트 vs 테스트 세트 | 학습에 쓰는 데이터 vs 성능 평가에 쓰는 처음 보는 데이터 |
| train_test_split vs 교차검증 | 한 번 나눠 평가 vs 여러 번 나눠 평균으로 평가 |
| 정밀도 vs 재현율 | "예측한 양성 중 진짜 비율" vs "실제 양성 중 잡아낸 비율" |
| R² vs RMSE | 설명력(높을수록 좋음) vs 평균 오차 크기(낮을수록 좋음) |
| 정확도만 보기 vs 다지표 보기 | 불균형에서 정확도는 오해를 부름, 재현율·F1 병행 필요 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [상관관계 분석](04-correlation-analysis.md)
- 다음에 이어지는 개념: 하이퍼파라미터 튜닝, 앙상블 모델
- 함께 보면 좋은 키워드: `과적합`, `혼동 행렬`, `데이터 누수`, `Pipeline`

## 셀프 체크

- [ ] 데이터를 훈련/테스트로 나누는 이유를 설명할 수 있다.
- [ ] K-fold 교차검증의 동작을 말할 수 있다.
- [ ] R²와 RMSE의 방향(높을수록/낮을수록 좋음)을 구분한다.
- [ ] 정밀도와 재현율의 차이를 예시로 설명할 수 있다.
- [ ] 불균형 데이터에서 정확도의 한계를 안다.

### 복습 질문 및 답변

**Q1. `random_state`는 왜 지정하나요?**

<details>
<summary>답</summary>

같은 분할·같은 결과를 재현하기 위해서입니다. 시드를 고정하면 실행할 때마다 동일한 훈련/테스트 분리가 됩니다.

</details>

**Q2. 교차검증이 단일 분리보다 나은 점은 무엇인가요?**

<details>
<summary>답</summary>

여러 번 나눠 평균을 내므로 한 번의 운 좋은(또는 나쁜) 분할에 흔들리지 않고, 성능을 더 안정적으로 추정합니다.

</details>

**Q3. 재현율이 특히 중요한 상황을 예로 들어 보세요.**

<details>
<summary>답</summary>

질병 진단이나 사기 탐지처럼 "실제 양성을 놓치면 손해가 큰" 상황입니다. 놓친 사례를 줄이는 것이 우선이므로 재현율을 중시합니다.

</details>

## 한 줄 정리

> 모델 평가는 처음 보는 데이터로, 여러 번 나눠, 문제에 맞는 지표로 봐야 진짜 실력을 알 수 있다.
