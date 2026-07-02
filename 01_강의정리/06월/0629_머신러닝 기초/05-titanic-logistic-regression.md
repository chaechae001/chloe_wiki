# 타이타닉 생존 예측과 로지스틱 회귀
> 로지스틱 회귀는 “생존할까?”처럼 결과가 0 또는 1인 문제에서, 먼저 1일 확률을 예측한 뒤 기준값으로 분류하는 모델이다.

`Titanic` · `EDA` · `전처리` · `Logistic Regression` · `Statsmodels` · `sigmoid` · `odds ratio` · `confusion matrix` · `precision` · `recall`

## 핵심요약

- 타이타닉 생존 예측은 생존 여부가 0/1인 이진 분류 문제다.
- 모델에 넣기 전 결측치 처리, 범주형 인코딩, 특성 선택이 필요하다.
- `age` 결측치는 중앙값으로 대체하고, `sex`는 male=0, female=1로 변환했다.
- NumPy로 인덱스를 섞어 학습 데이터 80%, 검증 데이터 20%로 나눴다.
- `statsmodels.api.Logit`으로 로지스틱 회귀 모델을 학습했다.
- 로지스틱 회귀는 선형 결합값을 시그모이드 함수에 넣어 0~1 사이 확률로 바꾼다.
- 예측 확률이 0.5 이상이면 생존으로 분류했다.
- 모델 평가는 정확도뿐 아니라 혼동 행렬, 정밀도, 재현율, F1-score를 함께 봐야 한다.

## 타이타닉 생존 분석 흐름

### 데이터 로드와 탐색

**1. 정의**  
탐색적 데이터 분석은 모델링 전에 데이터의 크기, 자료형, 결측치, 기초 통계, 주요 패턴을 확인하는 과정이다.

**2. 왜 필요한가?**  
데이터를 확인하지 않고 모델을 만들면 결측치, 문자형 변수, 이상치 때문에 오류가 나거나 잘못된 결과가 나올 수 있다.

**3. 예시**

```python
import seaborn as sns

titanic = sns.load_dataset('titanic')

print("First 5 rows:")
print(titanic.head())
print()

print("Shape:", titanic.shape)
print()

print("Data types:")
print(titanic.dtypes)
```

```python
print("Missing values per column:")
print(titanic.isnull().sum())

survival_rate = titanic['survived'].mean()
print(f"Overall survival rate: {survival_rate:.2%}")
print(f"Survived: {titanic['survived'].sum()} / Total: {len(titanic)}")
```

실행 결과 전체 생존율은 다음과 같이 확인되었다.

```text
Overall survival rate: 38.38%
Survived: 342 / Total: 891
```

**4. 헷갈리기 쉬운 점**  
`survived`의 평균이 생존율이 되는 이유는 값이 0과 1이기 때문이다. 1의 비율이 곧 평균이 된다.

**5. 한 줄 정리**  
EDA는 모델링 전에 데이터 상태와 기본 패턴을 확인하는 안전점검이다.

### 데이터 전처리

**1. 정의**  
전처리는 모델이 데이터를 이해할 수 있도록 결측치를 채우고, 문자를 숫자로 바꾸고, 사용할 열을 고르는 과정이다.

**2. 왜 필요한가?**  
대부분의 머신러닝 모델은 숫자 배열을 입력으로 받는다. 결측치나 문자열이 그대로 있으면 모델 학습이 어렵다.

**3. 예시**

```python
cols = ['survived', 'pclass', 'sex', 'age', 'sibsp', 'parch', 'fare']
df_model = titanic[cols].copy()

print("Missing values before cleaning:")
print(df_model.isnull().sum())
```

```text
survived      0
pclass        0
sex           0
age         177
sibsp         0
parch         0
fare          0
dtype: int64
```

```python
age_median = df_model['age'].median()
df_model['age'] = df_model['age'].fillna(age_median)

fare_median = df_model['fare'].median()
df_model['fare'] = df_model['fare'].fillna(fare_median)

df_model['sex'] = df_model['sex'].map({'male': 0, 'female': 1})
```

**4. 헷갈리기 쉬운 점**  
문자형 데이터를 숫자로 바꾼다고 해서 숫자의 크기 의미가 항상 생기는 것은 아니다. 여기서 `sex=1`이 `sex=0`보다 크다는 수학적 의미보다, 모델이 이해할 수 있는 코드값으로 바꾼 것이다.

**5. 한 줄 정리**  
전처리는 현실 데이터를 모델이 먹을 수 있는 숫자 음식으로 손질하는 과정이다.

### 학습/검증 데이터 분할

**1. 정의**  
전체 데이터를 학습용과 검증용으로 나누는 과정이다. 여기서는 80%는 학습, 20%는 검증에 사용했다.

**2. 왜 필요한가?**  
모델이 학습한 데이터만 잘 맞추는지, 처음 보는 데이터에도 잘 작동하는지 확인해야 하기 때문이다.

**3. 예시**

```python
np.random.seed(42)

n = len(df_model)
indices = np.arange(n)
np.random.shuffle(indices)

train_size = int(0.8 * n)
train_idx = indices[:train_size]
test_idx = indices[train_size:]

train_df = df_model.iloc[train_idx].reset_index(drop=True)
test_df = df_model.iloc[test_idx].reset_index(drop=True)

feature_cols = ['pclass', 'sex', 'age', 'sibsp', 'parch', 'fare']

X_train = train_df[feature_cols].values
y_train = train_df['survived'].values

X_test = test_df[feature_cols].values
y_test = test_df['survived'].values
```

실행 결과는 다음과 같다.

```text
Total samples: 891
Training samples: 712 (80%)
Testing samples: 179 (20%)

X_train shape: (712, 6)
y_train shape: (712,)
X_test shape: (179, 6)
y_test shape: (179,)
```

**4. 헷갈리기 쉬운 점**  
무작위 셔플 없이 앞에서부터 나누면 데이터 순서에 따른 편향이 생길 수 있다.

**5. 한 줄 정리**  
학습 데이터는 공부용, 검증 데이터는 모의고사용이다.

## 로지스틱 회귀

### 개념명

**1. 정의**  
로지스틱 회귀는 이진 분류를 위한 모델이다. 입력값들의 선형 결합을 만든 뒤, 시그모이드 함수로 0과 1 사이 확률로 변환한다.

**2. 왜 필요한가?**  
생존/사망, 이탈/유지, 구매/미구매처럼 결과가 둘 중 하나인 문제를 다룰 때 필요하다. 선형회귀는 결과가 음의 무한대부터 양의 무한대까지 나올 수 있어 0/1 분류에 직접 사용하기 어렵다.

**3. 예시**

```python
import statsmodels.api as sm

X_train_sm = sm.add_constant(X_train)
X_test_sm = sm.add_constant(X_test)

logit_model = sm.Logit(y_train, X_train_sm)
result = logit_model.fit()
```

**4. 헷갈리기 쉬운 점**  
로지스틱 회귀는 이름에 “회귀”가 들어가지만 분류에도 사용된다. 정확히는 사건이 발생할 확률을 예측하고, 그 확률을 기준으로 분류한다.

**5. 한 줄 정리**  
로지스틱 회귀는 0/1 결과를 직접 맞히기 전에 1일 확률을 계산하는 분류 모델이다.

### 계수와 오즈비 해석

**1. 정의**  
회귀계수는 각 특성이 생존 확률에 미치는 방향을 나타낸다. 오즈비는 계수를 지수 변환한 값으로, 생존 오즈가 몇 배로 변하는지 해석할 때 사용한다.

**2. 왜 필요한가?**  
모델이 왜 그런 예측을 했는지 설명하기 위해 필요하다. 정확도만 보면 모델의 판단 근거를 알 수 없다.

**3. 예시**

```python
feature_names = ['const (intercept)'] + feature_cols
coef_df = pd.DataFrame({
    'Feature': feature_names,
    'Coefficient': result.params,
    'Std Error': result.bse,
    'p-value': result.pvalues,
    'Odds Ratio': np.exp(result.params)
})
print(coef_df.round(4))
```

실행 결과는 다음과 같다.

```text
             Feature  Coefficient  Std Error  p-value  Odds Ratio
0  const (intercept)       2.4104     0.5390   0.0000     11.1385
1             pclass      -1.0959     0.1563   0.0000      0.3342
2                sex       2.6684     0.2208   0.0000     14.4166
3                age      -0.0446     0.0087   0.0000      0.9563
4              sibsp      -0.3986     0.1234   0.0012      0.6712
5              parch      -0.0591     0.1225   0.6293      0.9426
6               fare       0.0033     0.0026   0.2038      1.0033
```

**4. 헷갈리기 쉬운 점**  
계수가 양수면 생존 확률을 높이는 방향, 음수면 낮추는 방향이다. 단, 다른 변수가 고정되어 있다는 조건에서 해석해야 한다.

**5. 한 줄 정리**  
계수는 방향을, 오즈비는 영향의 배율을 해석하는 도구다.

## 코드로 보기 — 예측과 평가

```python
y_pred_prob = result.predict(X_test_sm)
y_pred = (y_pred_prob >= 0.5).astype(int)

print("First 10 predictions:")
print(f"Probabilities: {y_pred_prob[:10].round(3)}")
print(f"Predicted:     {y_pred[:10]}")
print(f"Actual:        {y_test[:10]}")
```

```text
First 10 predictions:
Probabilities: [0.287 0.064 0.274 0.608 0.828 0.065 0.884 0.305 0.261 0.276]
Predicted:     [0 0 0 1 1 0 1 0 0 0]
Actual:        [0 0 1 1 1 0 1 1 0 0]
```

```python
accuracy = np.mean(y_pred == y_test)
print(f"Model Accuracy on Test Set: {accuracy:.2%}")

TP = np.sum((y_pred == 1) & (y_test == 1))
TN = np.sum((y_pred == 0) & (y_test == 0))
FP = np.sum((y_pred == 1) & (y_test == 0))
FN = np.sum((y_pred == 0) & (y_test == 1))

precision = TP / (TP + FP) if (TP + FP) > 0 else 0
recall = TP / (TP + FN) if (TP + FN) > 0 else 0
f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0

print(f"Precision: {precision:.2%}")
print(f"Recall:    {recall:.2%}")
print(f"F1-Score:  {f1:.2%}")
```

실행 결과는 다음과 같다.

```text
Model Accuracy on Test Set: 77.65%

Confusion Matrix:
              Predicted 0    Predicted 1
Actual 0         TN= 93       FP= 21
Actual 1         FN= 19       TP= 46

Precision: 68.66%
Recall:    70.77%
F1-Score:  69.70%
```

**코드목적**  
예측 확률을 0/1 예측으로 바꾸고, 모델 성능을 여러 지표로 평가한다.

**해석**  
정확도는 77.65%였다. 실제 생존자 중 모델이 찾아낸 비율인 재현율은 70.77%, 생존이라고 예측한 사람 중 실제 생존한 비율인 정밀도는 68.66%였다.

**실무 연결**  
이탈 고객 탐지, 부정 거래 탐지, 질병 위험 예측처럼 “놓치면 안 되는 대상”이 있는 문제에서는 정확도보다 재현율이 더 중요할 수 있다.

## 모델 계수 저장과 재사용

```python
model_params = pd.DataFrame({
    'feature': ['const'] + feature_cols,
    'coefficient': np.asarray(result.params)
})

model_params.to_csv('/tmp/titanic_model_params.csv', index=False)
```

로지스틱 회귀는 계수만 있으면 같은 방식으로 예측 확률을 다시 계산할 수 있다.

```python
def predict_survival_prob(pclass, sex_enc, age, sibsp, parch, fare, coefs):
    linear = (coefs[0]
              + coefs[1] * pclass
              + coefs[2] * sex_enc
              + coefs[3] * age
              + coefs[4] * sibsp
              + coefs[5] * parch
              + coefs[6] * fare)
    prob = 1 / (1 + np.exp(-linear))
    return prob
```

실행 예시는 다음과 같다.

```text
Test prediction (1st class, female, age=25, fare=100):
Survival probability: 96.06%
Prediction: Survived

Test prediction (3rd class, male, age=35, fare=10):
Survival probability: 5.70%
Prediction: Not Survived
```

## 직접 해보기

1. 예측 임계값을 0.5에서 0.4로 낮추면 정밀도와 재현율이 어떻게 바뀔지 생각해보자.
2. `feature_cols`에서 `fare`를 제외하고 모델을 다시 만들면 성능이 어떻게 달라질지 확인해보자.
3. 성별 인코딩을 one-hot 방식으로 바꾸면 어떤 차이가 있을지 정리해보자.

## 헷갈리기 쉬운 포인트

| A | B | 차이 |
| --- | --- | --- |
| 선형회귀 | 로지스틱 회귀 | 연속값 예측 vs 0/1 확률 예측 |
| 확률 예측 | 최종 분류 | 0~1 값 출력 vs 임계값 기준 0/1 변환 |
| 정확도 | 재현율 | 전체 중 맞춘 비율 vs 실제 양성 중 찾아낸 비율 |
| 정밀도 | 재현율 | 예측 양성의 정확성 vs 실제 양성의 포착률 |
| 계수 | 오즈비 | 방향과 크기 vs 오즈 변화 배율 |

## 연결되는 개념

- 이전 글: [Matplotlib과 Seaborn으로 시각화하기](04-visualization-matplotlib-seaborn.md)
- 이전 글: [Pandas로 표 데이터 다루기](03-pandas-dataframe-basics.md)
- 더 찾아볼 키워드: sigmoid, logit, maximum likelihood estimation, ROC curve, threshold tuning

## 셀프 체크

- [ ] 로지스틱 회귀가 확률을 예측한다는 점을 설명할 수 있다.
- [ ] 결측치 처리와 범주형 인코딩이 필요한 이유를 설명할 수 있다.
- [ ] 학습/검증 데이터를 나누는 이유를 설명할 수 있다.
- [ ] 혼동 행렬의 TP, TN, FP, FN을 구분할 수 있다.
- [ ] 정밀도와 재현율의 차이를 설명할 수 있다.
- [ ] 계수와 오즈비를 해석할 수 있다.

**복습 질문 및 답변**

Q. 로지스틱 회귀는 왜 분류 모델로 사용할 수 있는가?  
A. 0~1 사이의 확률을 예측한 뒤, 임계값을 기준으로 0 또는 1로 분류할 수 있기 때문이다.

Q. `sm.add_constant()`는 왜 필요한가?  
A. statsmodels는 절편을 자동으로 넣지 않기 때문에 상수항을 직접 추가해야 한다.

Q. 정확도가 높아도 모델이 부족할 수 있는 이유는?  
A. 특정 클래스만 잘 맞추고 중요한 클래스를 놓칠 수 있기 때문이다. 그래서 혼동 행렬, 정밀도, 재현율을 함께 봐야 한다.

## 한 줄 정리

> 타이타닉 로지스틱 회귀 실습은 데이터 전처리, 분류 모델 학습, 확률 예측, 모델 평가, 계수 해석까지 머신러닝의 기본 흐름을 연결해 보여준다.
