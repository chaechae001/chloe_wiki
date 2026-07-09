# 회귀 파이프라인 실전 — 전처리 함수화부터 교차검증까지

> 파생변수·바이닝을 함수로 묶고, Pipeline으로 데이터 누수를 막으며 교차검증으로 성능을 검증하는 글입니다.

`회귀` · `Pipeline` · `ColumnTransformer` · `교차검증` · `데이터누수`

## 핵심요약

- 회귀는 연속적인 수치(가격, 팁 등)를 예측하는 문제이며, 이 글은 의사결정나무 회귀로 흐름을 익힌다.
- 파생변수와 바이닝은 반드시 **train/test 분리 이후**에 적용해야 데이터 누수를 막을 수 있다.
- 같은 전처리를 train·test·새 데이터에 똑같이 적용하려면 **함수화**가 필수다.
- 결측 대체·스케일링·인코딩은 **Pipeline** 안에 넣어, 교차검증 각 폴드에서 train 기준으로만 학습되게 한다.
- 성능은 RMSE·MAE·R²로 평가하고, **교차검증**으로 우연이 아닌 안정적 성능인지 확인한다.

## 1. 파이프라인과 데이터 누수 방지

### 1) 정의

Pipeline은 전처리(결측 대체, 스케일링, 인코딩)와 모델 학습을 하나의 흐름으로 묶는 도구입니다. 데이터 누수(Data Leakage)는 평가에 쓸 데이터의 정보가 학습 과정에 새어 들어가, 실제보다 성능이 좋아 보이게 만드는 문제를 말합니다.

### 2) 왜 필요한가

- 전처리를 전체 데이터에 미리 다 해놓고 나누면, test의 통계(중앙값·평균 등)가 학습에 섞여 누수가 생깁니다.
- Pipeline을 쓰면 교차검증의 매 폴드마다 오직 그 폴드의 train 데이터로만 전처리를 학습해 누수를 막습니다.
- 같은 전처리를 train·test·실서비스의 새 데이터에 일관되게 적용할 수 있어 실수를 줄입니다.

### 3) 핵심 흐름 재구성

```
1. 변수 선택   : 예측에 쓸 독립변수 일부를 고른다.
2. train/test 분리 : 반드시 이 시점에 먼저 나눈다.
3. 파생·바이닝  : 분리 이후에 만든다 (함수로 정의).
4. Pipeline 구성 :
   - 수치형: 결측 중앙값 대체 → StandardScaler
   - 범주형: 결측 최빈값 대체 → OneHotEncoder
   - ColumnTransformer로 두 흐름을 컬럼별로 연결
   - preprocessor + model을 하나의 Pipeline으로 결합
5. 교차검증    : KFold로 성능의 안정성 확인
6. 최종 학습·평가 : 전체 train으로 학습 후 test로 평가
```

특히 **분리 → 파생/바이닝 → Pipeline** 순서가 핵심입니다. 결측 대체나 스케일링을 함수 밖에서 전체 데이터에 미리 하면 누수가 생기므로, 이런 작업은 Pipeline 내부로 넘깁니다.

### 4) 쉬운 예시

시험 대비에 비유하면, 데이터 누수는 "모의고사 문제를 미리 보고 공부한 뒤 그 모의고사를 푸는 것"과 같습니다. 점수는 잘 나오지만 실제 실력은 아닙니다. Pipeline은 "공부할 때는 오직 교과서(train)만 보고, 모의고사(test)는 시험장에서 처음 본다"는 규칙을 자동으로 지켜줍니다.

### 5) 코드 예시

공개 데이터셋 `tips`로 팁(`tip`)을 예측하는 회귀 파이프라인을 구성합니다.

```python
import seaborn as sns, pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.tree import DecisionTreeRegressor

tips = sns.load_dataset('tips')
X = tips[['total_bill', 'size', 'day', 'time', 'smoker']].copy()
y = tips['tip'].copy()

# 2. 반드시 먼저 분리
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42)

# 3. 파생·바이닝은 분리 이후, 함수로 정의
def make_features(data):
    d = data.copy()
    d.loc[:, 'bill_per_person'] = d['total_bill'] / d['size']   # 파생변수
    d.loc[:, 'bill_band'] = pd.cut(                              # 바이닝
        d['total_bill'], bins=[0, 15, 30, 100],
        labels=['소액', '중간', '고액']).astype('object')
    return d[['total_bill', 'size', 'bill_per_person',
              'day', 'time', 'smoker', 'bill_band']]

X_tr, X_te = make_features(X_train), make_features(X_test)
```

같은 함수 하나로 train과 test에 동일한 전처리를 적용하는 것이 핵심입니다.

### 6) 헷갈리는 점

- 파생변수를 만들 때 쓴 원본 변수(예: `total_bill`)를 그대로 둘지 뺄지는 상황에 따라 다릅니다. 정보가 겹치면 다중공선성을 고려해 정리합니다.
- `OneHotEncoder(handle_unknown='ignore')`는 train에 없던 새 범주가 test에 나와도 에러 대신 모두 0으로 처리해 실서비스 안정성을 높입니다.
- 결측 대체를 함수 안에 넣으면 누수 위험이 생깁니다. 대체·스케일링은 Pipeline 내부로 넘기는 것이 안전합니다.

### 7) 한 줄 정리

> Pipeline은 전처리와 모델을 하나로 묶어, 교차검증에서도 누수 없이 일관된 전처리를 보장하는 안전장치입니다.

## 코드로 보기 — Pipeline 구성과 교차검증

```python
from sklearn.model_selection import KFold, cross_validate
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

num_f = ['total_bill', 'size', 'bill_per_person']
cat_f = ['day', 'time', 'smoker', 'bill_band']

# 수치형: 결측 중앙값 대체 → 표준화
num_t = Pipeline([('imp', SimpleImputer(strategy='median')),
                  ('sc', StandardScaler())])
# 범주형: 결측 최빈값 대체 → 원핫 인코딩
cat_t = Pipeline([('imp', SimpleImputer(strategy='most_frequent')),
                  ('oh', OneHotEncoder(handle_unknown='ignore'))])

pre = ColumnTransformer([('num', num_t, num_f), ('cat', cat_t, cat_f)])
model = DecisionTreeRegressor(max_depth=4, min_samples_leaf=10, random_state=42)
pipe = Pipeline([('pre', pre), ('model', model)])

# 5-Fold 교차검증
cv = KFold(n_splits=5, shuffle=True, random_state=42)
scoring = {'rmse': 'neg_root_mean_squared_error',
           'mae': 'neg_mean_absolute_error', 'r2': 'r2'}
cvr = cross_validate(pipe, X_tr, y_train, cv=cv,
                     scoring=scoring, return_train_score=True)

print('valid RMSE:', round(-cvr['test_rmse'].mean(), 3))  # 1.238
print('valid MAE :', round(-cvr['test_mae'].mean(), 3))   # 0.875
print('valid R2  :', round(cvr['test_r2'].mean(), 3))     # 0.173
print('train R2  :', round(cvr['train_r2'].mean(), 3))    # 0.497
```

평가지표의 의미:

```
RMSE = √( 평균( (실제 − 예측)² ) )   큰 오차에 더 큰 벌점 (단위는 원래 값과 같음)
MAE  = 평균( |실제 − 예측| )         평균적인 오차 크기 (직관적)
R²   = 1 − (오차 제곱합 ÷ 전체 분산)  모델이 분산을 설명하는 비율 (1에 가까울수록 좋음)
```

### 코드 목적

전처리와 모델을 하나로 묶은 뒤, 5-Fold 교차검증으로 성능이 우연이 아닌지, 과적합은 없는지 확인합니다.

### 코드 흐름

1. 수치형·범주형 각각의 전처리 흐름을 정의한다.
2. `ColumnTransformer`로 두 흐름을 컬럼별로 연결한다.
3. 전처리기와 모델을 하나의 Pipeline으로 결합한다.
4. `cross_validate`로 5개 폴드의 성능을 계산한다.
5. train 점수와 valid 점수를 비교해 과적합을 진단한다.

### 실행 결과 해석

이 예제에서는 train R²(0.497)가 valid R²(0.173)보다 눈에 띄게 높습니다. 학습 데이터에는 잘 맞지만 새로운 데이터에는 상대적으로 덜 맞는, 약한 과적합 경향을 보여줍니다. 실무에서는 이럴 때 `max_depth`를 더 줄이거나, 더 나은 특성을 추가하거나, 다른 모델을 비교하는 식으로 개선합니다. 최종적으로 test에서 R²=0.515가 나왔는데, 팁이라는 변동이 큰 값을 단순 모델로 예측한 것 치고는 절반가량의 분산을 설명한 셈입니다.

참고로 특성 중요도를 보면 `total_bill`(0.93)이 압도적입니다. 계산서 금액이 팁을 결정하는 가장 큰 요인이라는, 상식과 일치하는 결과입니다.

### 실무 연결

주택 가격 예측, 수요량 예측, 광고비 대비 매출 예측 등 "연속적인 수치"를 맞히는 문제 전반에 이 구조가 그대로 쓰입니다. 특히 교차검증과 Pipeline은 캐글 같은 대회뿐 아니라 실서비스 모델 검증의 표준 절차입니다. 새 데이터 1건을 예측할 때도 반드시 같은 `make_features` 함수와 학습된 `pipe`를 거쳐야 학습 때와 동일한 전처리가 보장됩니다.

## 직접 해보기

1. 파생변수와 바이닝을 train/test 분리 **이전에** 하면 어떤 문제가 생기나요?
2. RMSE와 MAE 중, 큰 오차 하나에 더 민감하게 반응하는 지표는 무엇이고 왜 그런가요?
3. 교차검증에서 train R²는 0.95인데 valid R²는 0.40이 나왔습니다. 무슨 상황이고, 어떻게 대응할 수 있을까요?

<details>
<summary>정답 보기</summary>

1. 데이터 누수가 생깁니다. 예컨대 결측 대체나 스케일링을 전체 데이터로 먼저 하면 test의 통계 정보가 train 전처리에 섞여 들어가, 검증 성능이 실제보다 부풀려집니다. 그래서 분리를 먼저 하고, 대체·스케일링은 Pipeline 내부로 넘겨 각 폴드의 train 기준으로만 학습되게 합니다.

2. RMSE입니다. 오차를 제곱해서 평균 내므로, 큰 오차 하나가 제곱되면서 훨씬 크게 반영됩니다. 반면 MAE는 절댓값의 평균이라 모든 오차를 동등하게 다룹니다. 큰 오차를 특히 피하고 싶다면 RMSE를 주목합니다.

3. 과적합 상황입니다. 모델이 학습 데이터의 세부(잡음까지)를 외워서 train 점수는 높지만, 새 데이터에는 일반화되지 않아 valid 점수가 낮습니다. 대응으로는 나무 깊이(`max_depth`)를 줄이거나 `min_samples_leaf`를 늘려 모델을 단순화하고, 데이터를 늘리거나 더 유효한 특성을 추가하는 방법이 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 분류 vs 회귀 | 범주(생존/사망)를 맞히면 분류, 연속 수치(가격/팁)를 맞히면 회귀 |
| RMSE vs MAE | 큰 오차에 벌점을 크게 주면 RMSE, 오차를 동등하게 평균 내면 MAE |
| 데이터 분리 전 vs 후 전처리 | 파생·바이닝·대체를 분리 후 train 기준으로 해야 누수 방지 |
| train score vs valid score | 학습 데이터 성적이 train, 검증(모의고사) 성적이 valid — 둘의 격차가 과적합 신호 |

## 연결되는 개념

- 이전에 알면 좋은 개념: 정상성과 확률보행
- 다음에 이어지는 개념: 하이퍼파라미터 튜닝, 앙상블 모델
- 함께 보면 좋은 키워드: `Pipeline`, `cross_validate`, `과적합`

## 셀프 체크

- [ ] 데이터 누수가 무엇이고 왜 위험한지 설명할 수 있다.
- [ ] 파생·바이닝을 분리 이후에 해야 하는 이유를 안다.
- [ ] ColumnTransformer로 수치형·범주형을 나눠 전처리할 수 있다.
- [ ] RMSE·MAE·R²의 차이를 설명할 수 있다.
- [ ] train/valid 점수 격차로 과적합을 진단할 수 있다.

### 복습 질문 및 답변

**Q1. 전처리를 왜 Pipeline 안에 넣나요? 밖에서 하면 안 되나요?**

<details>
<summary>답</summary>

밖에서 전체 데이터에 전처리를 하면, 교차검증에서 각 폴드의 test 정보가 이미 전처리에 반영되어 누수가 생깁니다. Pipeline 안에 넣으면 매 폴드마다 그 폴드의 train으로만 대체·스케일링을 학습하므로 누수를 막고, 실제에 가까운 성능을 얻습니다.

</details>

**Q2. 교차검증은 단순히 한 번 나눠 평가하는 것과 무엇이 다른가요?**

<details>
<summary>답</summary>

한 번만 나누면 그 분할이 우연히 쉬웠는지 어려웠는지에 따라 성능이 크게 흔들립니다. 교차검증은 데이터를 여러 폴드로 나눠 번갈아 평가하고 평균을 내므로, 특정 분할에 운 좋게 맞은 것이 아닌 안정적인 성능을 추정할 수 있습니다.

</details>

**Q3. `handle_unknown='ignore'`는 왜 필요한가요?**

<details>
<summary>답</summary>

학습 때 보지 못한 새 범주가 실제 데이터나 test에 나타날 수 있기 때문입니다. 이 옵션이 없으면 그럴 때 에러가 나서 예측이 중단됩니다. `ignore`로 두면 새 범주를 모두 0으로 처리해 예측을 이어갈 수 있어 실서비스에서 안정적입니다.

</details>

## 한 줄 정리

> 회귀 파이프라인은 분리 → 함수화 → Pipeline → 교차검증 순서로, 누수를 막고 성능을 안정적으로 검증하는 실전 절차입니다.
