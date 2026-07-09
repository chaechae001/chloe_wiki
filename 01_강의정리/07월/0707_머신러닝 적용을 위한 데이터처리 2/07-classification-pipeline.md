# 분류 종합 파이프라인 — 층화분리부터 튜닝·평가까지

> 좋은 분류 모델의 절반은 알고리즘이 아니라 "검증 설계와 순서"입니다. 이 글은 앞선 개념들을 하나의 흐름으로 연결합니다.

`Pipeline` · `StratifiedKFold` · `GridSearchCV` · `RandomForest` · `ROC-AUC`

## 핵심요약

- 분류에서는 학습/테스트 분리와 교차검증 모두 클래스 비율을 유지하는 층화(stratify) 방식을 쓴다.
- 결측치 대체·인코딩·스케일링은 반드시 분리 이후, Pipeline 안에서 수행해 누수를 막는다.
- StratifiedKFold 교차검증으로 fold마다 클래스 비율을 유지하며 안정적으로 성능을 추정한다.
- GridSearchCV는 하이퍼파라미터 조합을 교차검증으로 탐색하며, 튜닝도 학습 데이터 안에서만 한다.
- 테스트 데이터는 마지막 단 한 번, 최종 평가에만 사용한다.

## 1. 층화 추출 — 분류의 기본기

### 1) 정의

층화 추출(stratified sampling)은 데이터를 나눌 때 원본의 클래스 비율을 각 분할에도 그대로 유지하는 방식입니다. `train_test_split(..., stratify=y)`로 적용합니다.

### 2) 왜 필요한가

불균형하거나 클래스가 여러 개인 분류에서, 무작위 분할은 우연히 특정 클래스가 한쪽에 몰릴 수 있습니다. 그러면 평가가 불안정해집니다. 층화를 쓰면 train과 test의 클래스 비율이 원본과 비슷하게 유지됩니다.

### 3) 코드 예시 (seaborn titanic)

```python
import seaborn as sns
from sklearn.model_selection import train_test_split

t = sns.load_dataset('titanic')
num = ['pclass', 'age', 'sibsp', 'parch', 'fare']
cat = ['sex', 'embarked', 'who', 'alone']
X = t[num + cat].copy()
for c in cat:
    X[c] = X[c].astype('object')
y = t['survived']

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
print("타깃 분포:", dict(y.value_counts(normalize=True).round(3).sort_index()))
```

실행 결과: `타깃 분포: {0: 0.616, 1: 0.384}` — 생존/사망 비율이 train과 test에 비슷하게 유지됩니다.

### 4) 한 줄 정리

> 분류에서는 층화 분리로 클래스 비율을 지켜야 평가가 흔들리지 않는다.

## 2. Pipeline + ColumnTransformer

### 1) 정의

`ColumnTransformer`는 수치형·범주형처럼 성격이 다른 열에 서로 다른 전처리를 적용하고, `Pipeline`은 그 전처리와 모델을 하나로 묶는 도구입니다.

### 2) 왜 필요한가

- 수치형은 결측치 중앙값 대체 후 스케일링, 범주형은 최빈값 대체 후 원-핫 인코딩처럼 열마다 처리가 다릅니다.
- 이를 Pipeline으로 묶으면 교차검증의 각 fold에서 전처리가 학습 fold에만 fit되어, 누수가 자동으로 차단됩니다(→ [데이터 누수와 파이프라인](04-data-leakage.md)).

### 3) 코드 예시

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier

pre = ColumnTransformer([
    ('num', Pipeline([('imp', SimpleImputer(strategy='median')),
                      ('sc', StandardScaler())]), num),
    ('cat', Pipeline([('imp', SimpleImputer(strategy='most_frequent')),
                      ('oh', OneHotEncoder(drop='first', handle_unknown='ignore'))]), cat),
])
pipe = Pipeline([('pre', pre), ('model', RandomForestClassifier(random_state=42))])
```

### 4) 한 줄 정리

> ColumnTransformer로 열별 전처리를, Pipeline으로 전처리+모델을 묶어 누수 없이 재현 가능하게 만든다.

## 3. StratifiedKFold 교차검증

### 1) 정의

StratifiedKFold는 데이터를 K개로 나눠 번갈아 검증하되, 각 fold의 클래스 비율을 유지하는 교차검증 방식입니다.

### 2) 코드 예시

```python
from sklearn.model_selection import StratifiedKFold, cross_validate

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
res = cross_validate(pipe, X_tr, y_tr, cv=cv,
                     scoring=['accuracy', 'f1', 'roc_auc', 'recall'],
                     return_train_score=True)

for m in ['accuracy', 'f1', 'roc_auc', 'recall']:
    print(f"{m:9s} train={res['train_'+m].mean():.3f}  valid={res['test_'+m].mean():.3f}")
```

실행 결과:

```
accuracy  train=0.982  valid=0.809
f1        train=0.977  valid=0.748
roc_auc   train=0.998  valid=0.869
recall    train=0.970  valid=0.740
```

### 3) 결과 해석

train 점수가 valid보다 뚜렷이 높습니다(예: accuracy 0.982 vs 0.809). 이는 기본 설정 RandomForest가 학습 데이터에 다소 과적합되어 있다는 신호로, 하이퍼파라미터 튜닝으로 완화할 여지를 보여줍니다.

### 4) 한 줄 정리

> StratifiedKFold는 클래스 비율을 유지한 채 여러 번 검증해 성능을 안정적으로 추정한다.

## 4. GridSearchCV 하이퍼파라미터 튜닝

### 1) 정의

GridSearchCV는 지정한 하이퍼파라미터 조합을 모두 교차검증으로 시험해, 가장 좋은 조합을 찾는 도구입니다.

### 2) 코드 예시

```python
from sklearn.model_selection import GridSearchCV

grid = {
    'model__n_estimators': [100, 200],       # 나무 개수
    'model__max_depth': [3, 5, None],        # 최대 깊이(가지치기)
    'model__min_samples_leaf': [1, 3, 5],    # 잎 노드 최소 샘플
    'model__max_features': ['sqrt', 'log2'], # 분할 시 고려할 특성 수
}
gs = GridSearchCV(pipe, grid, scoring='roc_auc', cv=cv, n_jobs=-1)
gs.fit(X_tr, y_tr)

print("Best ROC-AUC(cv):", round(gs.best_score_, 3))
print("Best params:", {k.replace('model__', ''): v for k, v in gs.best_params_.items()})
```

실행 결과:

```
Best ROC-AUC(cv): 0.882
Best params: {'max_depth': None, 'max_features': 'sqrt', 'min_samples_leaf': 3, 'n_estimators': 100}
```

### 3) 결과 해석

파라미터 접두어 `model__`은 "Pipeline 안의 model 단계에 전달하라"는 의미입니다. 튜닝은 오직 학습 데이터(`X_tr`)로만 진행하며, 테스트 데이터는 여기 관여하지 않습니다. 최적 조합으로 교차검증 ROC-AUC가 0.882까지 올랐습니다.

### 4) 한 줄 정리

> GridSearchCV는 학습 데이터 안에서 조합을 탐색하며, 테스트 데이터는 튜닝에 절대 쓰지 않는다.

## 코드로 보기 — 최종 테스트 평가

`GridSearchCV`는 최적 조합으로 전체 학습 데이터를 다시 학습한 모델을 `best_estimator_`에 담아 둡니다. 이제 처음 떼어 둔 테스트 데이터로 마지막 평가를 합니다.

```python
from sklearn.metrics import classification_report, roc_auc_score

best = gs.best_estimator_
y_pred = best.predict(X_te)
y_proba = best.predict_proba(X_te)[:, 1]

print(classification_report(y_te, y_pred, target_names=['사망', '생존']))
print("Test ROC-AUC:", round(roc_auc_score(y_te, y_proba), 3))
```

실행 결과:

```
              precision    recall  f1-score   support

          사망       0.81      0.92      0.86       110
          생존       0.84      0.67      0.74        69

    accuracy                           0.82       179
   macro avg       0.83      0.79      0.80       179
weighted avg       0.82      0.82      0.82       179

Test ROC-AUC: 0.839
```

### 실행 결과 해석

- 전체 정확도는 0.82지만, 클래스별로 보면 "생존"의 재현율이 0.67로 "사망"(0.92)보다 낮습니다. 생존자를 놓치는 경우가 상대적으로 많다는 뜻입니다.
- ROC-AUC 0.839는 두 클래스를 구분하는 전반적 능력이 준수함을 보여줍니다.
- 교차검증 valid 점수(예: roc_auc 0.869)와 테스트 점수(0.839)가 비슷하면, 검증 설계가 신뢰할 만하다는 신호입니다. 두 점수가 크게 벌어지면 누수나 과적합을 의심해야 합니다.

### 실무 연결

이탈 예측, 승인/거절 심사, 불량 판정 등 대부분의 실무 분류 문제가 이 흐름(층화 분리 → Pipeline 전처리 → 교차검증 → 튜닝 → 최종 평가)을 따릅니다. 관심 클래스를 놓치는 비용이 크다면 `scoring`을 recall이나 f1로 바꿔 튜닝 기준을 조정합니다.

## 직접 해보기

1. `scoring='roc_auc'`로 튜닝했는데, 실무에서 소수 클래스 검출이 최우선이라면 무엇을 바꾸겠는가?
2. 교차검증 valid 점수는 0.87인데 테스트 점수가 0.70으로 크게 낮다. 무엇을 의심해야 하나?
3. `param_grid`의 키에 `model__`을 붙이는 이유는?

<details>
<summary>정답 보기</summary>

1. `scoring`을 `'recall'` 또는 `'f1'`로 바꾼다. ROC-AUC는 전반적 구분력을 보지만, 소수 클래스를 놓치지 않는 것이 핵심이면 재현율/ F1을 튜닝 기준으로 삼아야 한다.
2. 데이터 누수나 과적합을 의심한다. 검증 점수만 높고 테스트가 낮다면, 전처리가 분리 전에 이뤄졌거나(누수) 모델이 학습 데이터에 과적합됐을 가능성이 크다.
3. Pipeline 안의 특정 단계(여기서는 `model`)에 파라미터를 전달하기 위한 규칙이다. `단계이름__파라미터명` 형식으로 지정한다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| KFold vs StratifiedKFold | 단순 K분할 vs 클래스 비율을 유지한 K분할(분류에 적합) |
| validation vs test | 튜닝·검증에 반복 사용 vs 최종 평가에 단 한 번 사용 |
| best_score_ vs 테스트 점수 | 교차검증 최적 점수(학습 데이터 기반) vs 미사용 테스트 데이터 점수 |
| GridSearch vs RandomSearch | 모든 조합 전수 탐색 vs 일부 조합만 무작위 탐색(빠름) |
| accuracy vs ROC-AUC | 임계값 0.5 기준 맞힌 비율 vs 임계값 전반의 구분력 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [데이터 누수와 파이프라인](04-data-leakage.md), [모델별 특성 엔지니어링](06-feature-engineering.md)
- 함께 보면 좋은 키워드: `stratify`, `cross_validate`, `best_estimator_`, `classification_report`

## 셀프 체크

- [ ] 층화 분리가 왜 필요한지 안다.
- [ ] ColumnTransformer로 열별 전처리를 구성할 수 있다.
- [ ] StratifiedKFold 교차검증의 목적을 안다.
- [ ] GridSearchCV의 튜닝이 학습 데이터 안에서만 이뤄져야 함을 안다.
- [ ] 테스트 데이터를 언제 사용해야 하는지 안다.

### 복습 질문 및 답변

**Q1. 전처리를 train/test 분리 전에 하면 안 되는 이유는?**

<details>
<summary>답</summary>

테스트 데이터의 통계(평균·중앙값 등)가 전처리 기준에 섞여 학습에 유입되는 데이터 누수가 발생한다. 분리를 먼저 하고, 전처리는 Pipeline 안에서 학습 데이터로만 fit해야 한다.

</details>

**Q2. 교차검증 점수와 최종 테스트 점수가 비슷하면 무엇을 뜻하나?**

<details>
<summary>답</summary>

검증 설계가 신뢰할 만하다는 신호다. 두 점수가 크게 벌어지면 누수나 과적합을 의심해야 한다.

</details>

**Q3. GridSearchCV의 best_estimator_는 어떤 모델인가?**

<details>
<summary>답</summary>

교차검증에서 가장 좋은 하이퍼파라미터 조합으로, 전체 학습 데이터를 다시 학습시킨 모델이다. 이 모델로 처음 떼어 둔 테스트 데이터를 최종 평가한다.

</details>

## 한 줄 정리

> 좋은 분류 파이프라인은 층화 분리로 시작해 Pipeline 전처리·교차검증·튜닝을 거치며, 테스트 데이터는 마지막 한 번만 사용한다.
