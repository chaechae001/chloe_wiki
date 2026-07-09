# 하이퍼파라미터 튜닝·교차검증·파이프라인·데이터 누수

> 모델의 다이얼을 제대로 맞추는 법, 그리고 조용히 성능을 부풀리는 함정을 피하는 법.

`하이퍼파라미터` · `GridSearch` · `교차검증` · `파이프라인` · `데이터누수`

## 핵심요약

- 하이퍼파라미터는 학습 전에 사람이 정하는 설정값으로, 성능을 크게 좌우한다.
- 교차검증은 데이터를 여러 조각으로 나눠 돌아가며 검증해 성능을 안정적으로 추정한다.
- GridSearch/RandomSearch는 하이퍼파라미터 조합을 탐색해 좋은 값을 찾는다.
- 파이프라인은 전처리와 모델을 하나로 묶어 데이터 누수를 구조적으로 막는다.
- 데이터 누수는 검증·미래 정보가 학습에 새어 들어 성능을 부풀리는 치명적 실수다.

---

## 1. 하이퍼파라미터와 교차검증

### 1) 정의

하이퍼파라미터(hyperparameter)는 학습으로 자동 결정되는 값(파라미터)과 달리, 학습을 시작하기 전 사람이 지정하는 설정이다. 트리 깊이, 학습률, 이웃 수 K 등이 여기 속한다. 교차검증(Cross Validation)은 데이터를 K개 조각으로 나눠, 하나를 검증용으로 두고 나머지로 학습하는 과정을 K번 반복해 평균 성능을 구하는 방법이다.

### 2) 왜 필요한가

한 번의 train/test 분할은 우연에 흔들린다. 어쩌다 쉬운 데이터가 test로 가면 성능이 부풀려 보인다. 교차검증은 여러 분할의 평균을 내 이 우연을 줄이고, 하이퍼파라미터 선택을 더 믿을 만하게 만든다.

### 3) 핵심 흐름 재구성

- 데이터를 K개 폴드로 나눈다(예: K=5).
- 각 폴드를 한 번씩 검증용으로 쓰며 K번 학습·평가한다.
- K개 점수의 평균을 그 설정의 성능으로 본다.
- 분류에서는 클래스 비율을 유지하는 StratifiedKFold를 쓴다.

### 4) 쉬운 예시

한 번의 모의고사 점수로 실력을 판단하면 그날 컨디션에 좌우된다. 다섯 번 시험을 보고 평균을 내면 훨씬 안정적이다. 교차검증이 바로 이 "여러 번 보고 평균 내기"다.

### 5) 코드 예시

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score
from sklearn.ensemble import RandomForestClassifier

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
model = RandomForestClassifier(n_estimators=300, random_state=42)

scores = cross_val_score(model, X_tr, y_tr, cv=cv, scoring='roc_auc')
print("폴드별 AUC:", scores.round(3))
print("평균 AUC:", scores.mean().round(3))
```

폴드별 점수의 편차가 크면 모델이 데이터 분할에 민감하다는 신호다.

### 6) 헷갈리는 점

- 파라미터(가중치)와 하이퍼파라미터(설정)는 다르다. 전자는 학습이 정하고, 후자는 사람이 정한다.
- 교차검증은 train 데이터 안에서만 한다. test는 마지막 최종 평가에만 쓴다.

### 7) 한 줄 정리

> 교차검증은 여러 분할의 평균으로 성능을 안정적으로 추정해 하이퍼파라미터 선택을 돕는다.

---

## 2. GridSearch로 튜닝하기

### 1) 정의

GridSearch는 하이퍼파라미터 후보들을 격자(grid)로 나열해 모든 조합을 교차검증으로 시험하고, 가장 좋은 조합을 고르는 방법이다. RandomSearch는 조합을 무작위로 일부만 뽑아 더 빠르게 탐색한다.

### 2) 왜 필요한가

하이퍼파라미터를 감으로 정하면 성능을 놓치기 쉽다. 체계적으로 탐색하면 더 나은 설정을 재현 가능하게 찾을 수 있다.

### 3) 핵심 흐름 재구성

- 탐색할 하이퍼파라미터와 후보 값을 정의한다.
- 각 조합마다 교차검증으로 점수를 매긴다.
- 최고 점수 조합(`best_params_`)을 고른다.
- 그 설정으로 전체 train에 다시 학습해 test로 최종 평가한다.

조합이 많으면 계산이 폭발적으로 늘어난다(조합 수 × 폴드 수). 이때 RandomSearch나 좁은 범위 탐색이 유용하다.

### 4) 쉬운 예시

커피 레시피를 맞춘다고 하자. 원두 양 3종 × 물 온도 3종 = 9가지 조합을 모두 내려 마셔보고 가장 맛있는 조합을 고르는 것이 GridSearch다. 조합이 너무 많으면 몇 개만 무작위로 시음(RandomSearch)한다.

### 5) 코드 예시

```python
from sklearn.model_selection import GridSearchCV, StratifiedKFold

grid = {'model__n_estimators': [100, 300],
        'model__max_depth': [3, 5, None]}
cv = StratifiedKFold(5, shuffle=True, random_state=42)

gs = GridSearchCV(pipe, grid, cv=cv, scoring='roc_auc', n_jobs=-1)
gs.fit(X_tr, y_tr)
print("best params:", gs.best_params_)
print("best CV roc_auc:", round(gs.best_score_, 3))
```

`model__` 접두어는 파이프라인 안의 `model` 단계에 값을 전달하겠다는 표기다.

### 6) 헷갈리는 점

- GridSearch의 `best_score_`는 교차검증 점수이지 test 점수가 아니다. 최종 성능은 반드시 별도 test로 확인한다.
- 후보를 너무 넓게 잡으면 시간이 급증한다. 좁게 시작해 점점 조이는 편이 실용적이다.

### 7) 한 줄 정리

> GridSearch는 조합을 교차검증으로 비교해 좋은 하이퍼파라미터를 체계적으로 찾는다.

---

## 3. 파이프라인과 데이터 누수

### 1) 정의

데이터 누수(Data Leakage)는 학습 시점에 알 수 없어야 할 정보(검증·test·미래 데이터)가 학습에 새어 들어가 성능이 실제보다 부풀려지는 현상이다. 파이프라인(Pipeline)은 전처리와 모델을 하나로 묶어, 이 누수를 구조적으로 방지하는 도구다.

### 2) 왜 필요한가

누수가 있으면 검증에서는 훌륭해 보이던 모델이 실제 배포에서 무너진다. 발견하기도 어렵고 피해가 크다. 파이프라인은 전처리가 항상 train에만 fit되도록 강제해 이 위험을 줄인다.

### 3) 핵심 흐름 재구성

대표적 누수 유형:

- 스케일러·인코더를 전체 데이터에 `fit`(test 정보 반영)
- 결측치를 전체 평균으로 채움
- 분할 전에 오버샘플링·중복을 만들어 같은 행이 train/test에 동시 존재
- 정답(타깃)에서 파생된 특성을 무심코 사용

파이프라인은 교차검증의 매 폴드에서 전처리를 그 폴드의 train에만 fit하므로, 검증 조각으로의 누수를 막는다.

### 4) 쉬운 예시

시험 전날 문제지를 미리 본 학생을 생각하자. 시험 점수는 높지만 실력은 아니다. 데이터 누수는 모델이 정답을 미리 훔쳐본 상태와 같다.

### 5) 코드 예시 — 누수가 성능을 부풀리는 것 확인

```python
import seaborn as sns
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

df = sns.load_dataset('titanic')[
    ['survived','age','fare','pclass','sex','sibsp','parch']].dropna().copy()
df['sex'] = (df['sex'] == 'female').astype(int)
X = df[['age','fare','pclass','sex','sibsp','parch']]
y = df['survived']

# 잘못: 분할 '전'에 데이터를 복제 → 같은 행이 train/test 양쪽에 존재
Xd = pd.concat([X, X], ignore_index=True)
yd = pd.concat([y, y], ignore_index=True)
Xtr, Xte, ytr, yte = train_test_split(Xd, yd, test_size=0.3, random_state=42)
m = RandomForestClassifier(random_state=42).fit(Xtr, ytr)
print(f"[누수有] test acc={accuracy_score(yte, m.predict(Xte)):.3f}")

# 올바름: 분할 '후'에만 처리
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)
m = RandomForestClassifier(random_state=42).fit(Xtr, ytr)
print(f"[누수無] test acc={accuracy_score(yte, m.predict(Xte)):.3f}")
```

실행 결과:

```text
[누수有] test acc=0.916
[누수無] test acc=0.800
```

분할 전에 데이터를 복제했더니 test에 학습된 행과 똑같은 행이 섞여 정확도가 0.916으로 부풀려졌다. 올바른 방식(0.800)이 진짜 성능이다.

### 6) 헷갈리는 점

- 스케일링만의 누수는 종종 성능 차이가 작다. 하지만 오버샘플링·타깃 인코딩·중복처럼 정답이나 미래가 새면 차이가 극적으로 커진다.
- 파이프라인을 쓰면 이런 실수를 구조적으로 막을 수 있어, 습관적으로 파이프라인을 쓰는 편이 안전하다.

### 7) 한 줄 정리

> 데이터 누수는 성능을 조용히 부풀리는 함정이며, 파이프라인은 전처리를 train에만 묶어 이를 막는다.

---

## 코드로 보기 — 파이프라인 + GridSearch로 안전하게 튜닝

```python
import seaborn as sns
from sklearn.model_selection import train_test_split, GridSearchCV, StratifiedKFold
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, roc_auc_score

df = sns.load_dataset('titanic')[
    ['survived','age','fare','pclass','sex','embarked','sibsp','parch']].copy()
X = df.drop(columns='survived')
y = df['survived']
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

num = ['age','fare','sibsp','parch']
cat = ['pclass','sex','embarked']
pre = ColumnTransformer([
    ('num', Pipeline([('imp', SimpleImputer(strategy='median')),
                      ('sc', StandardScaler())]), num),
    ('cat', Pipeline([('imp', SimpleImputer(strategy='most_frequent')),
                      ('oh', OneHotEncoder(handle_unknown='ignore'))]), cat),
])
pipe = Pipeline([('pre', pre), ('model', RandomForestClassifier(random_state=42))])

grid = {'model__n_estimators': [100, 300], 'model__max_depth': [3, 5, None]}
cv = StratifiedKFold(5, shuffle=True, random_state=42)
gs = GridSearchCV(pipe, grid, cv=cv, scoring='roc_auc', n_jobs=-1)
gs.fit(X_tr, y_tr)

print("best params:", gs.best_params_)
print("best CV roc_auc:", round(gs.best_score_, 3))
print("test acc:", round(accuracy_score(y_te, gs.predict(X_te)), 3))
print("test auc:", round(roc_auc_score(y_te, gs.predict_proba(X_te)[:, 1]), 3))
```

### 코드 목적

전처리와 모델을 파이프라인으로 묶어 누수를 막으면서, GridSearch로 하이퍼파라미터를 튜닝하고 test로 정직하게 최종 평가하기 위한 코드다.

### 코드 흐름

1. 수치형·범주형을 각각 결측치 처리·스케일·인코딩하는 전처리를 정의한다.
2. 전처리와 모델을 하나의 파이프라인으로 묶는다.
3. 교차검증 기반 GridSearch로 최적 조합을 찾는다.
4. 최적 모델을 test로 최종 평가한다.

### 실행 결과 해석

```text
best params: {'model__max_depth': 5, 'model__n_estimators': 300}
best CV roc_auc: 0.870
test acc: 0.804
test auc: 0.844
```

교차검증 AUC(0.870)와 test AUC(0.844)가 비슷하게 나와, 튜닝이 특정 분할에 과하게 맞춰지지 않았음을 보여준다. 두 값이 크게 벌어졌다면 누수나 과적합을 의심해야 한다.

### 실무 연결

실제 모델 개발에서는 이 파이프라인 패턴이 표준이다. 전처리부터 모델까지 하나로 묶어야 배포 시에도 같은 순서로 재현되고, 누수 없이 안전하게 튜닝할 수 있다.

---

## 직접 해보기

1. 파라미터와 하이퍼파라미터의 차이를 한 문장으로 설명하라.
2. 5-폴드 교차검증에서 각 폴드는 몇 번 학습에, 몇 번 검증에 쓰이는가?
3. 스케일러를 전체 데이터에 fit하면 어떤 문제가 생기며, 어떻게 막는가?

<details>
<summary>정답 보기</summary>

1. 파라미터는 학습이 자동으로 정하는 값(예: 회귀계수)이고, 하이퍼파라미터는 학습 전에 사람이 정하는 설정(예: 트리 깊이)이다.
2. 각 폴드는 검증에 1번, 학습에 4번 쓰인다(자기 차례에 검증, 나머지 4번은 다른 폴드가 검증일 때 학습에 포함).
3. test 정보(평균·표준편차)가 스케일 기준에 반영되는 데이터 누수가 생긴다. train에만 fit하거나 파이프라인으로 묶어 매 폴드의 train에만 fit되게 하면 막을 수 있다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 파라미터 vs 하이퍼파라미터 | 학습이 정하면 파라미터, 사람이 정하면 하이퍼파라미터 |
| GridSearch vs RandomSearch | 모든 조합 탐색이 Grid, 무작위 일부 탐색이 Random |
| CV 점수 vs test 점수 | CV는 튜닝용 추정치, test는 최종 성능 확인용 |
| 전체 fit vs train만 fit | 전체 fit은 누수 위험, train만 fit이 올바름 |

## 연결되는 개념

- 이전에 알면 좋은 개념: 그래디언트 부스팅 3형제 비교
- 다음에 이어지는 개념: 스태킹 앙상블로 모델 결합하기
- 함께 보면 좋은 키워드: `StratifiedKFold`, `ColumnTransformer`, `best_params_`

## 셀프 체크

- [ ] 하이퍼파라미터가 무엇인지 안다.
- [ ] 교차검증이 왜 더 안정적인지 설명할 수 있다.
- [ ] GridSearch의 작동 방식을 안다.
- [ ] 데이터 누수의 대표 유형을 예로 들 수 있다.
- [ ] 파이프라인이 누수를 막는 원리를 안다.

### 복습 질문 및 답변

**Q1. 왜 test 데이터로는 하이퍼파라미터를 고르면 안 되는가?**

<details>
<summary>답</summary>

test로 튜닝하면 그 test에 맞춘 설정을 고르게 되어 test가 더 이상 "본 적 없는 데이터"가 아니게 된다. 최종 성능이 부풀려져 배포 후 실제 성능과 어긋난다. 그래서 튜닝은 교차검증(train 내부)으로 하고 test는 마지막 확인용으로만 남긴다.

</details>

**Q2. 교차검증 점수의 폴드별 편차가 크다면 무엇을 의심해야 하나?**

<details>
<summary>답</summary>

모델이 데이터 분할에 민감하거나 데이터가 작고 불안정하다는 신호다. 데이터를 늘리거나, 더 단순한 모델·규제 강화를 검토할 수 있다.

</details>

**Q3. 파이프라인이 없어도 누수를 막을 수 있는가?**

<details>
<summary>답</summary>

수동으로 train에만 fit하고 각 폴드마다 전처리를 반복하면 가능하지만 실수하기 쉽다. 파이프라인은 이 과정을 자동화해 교차검증·GridSearch에서 누수를 구조적으로 차단하므로 훨씬 안전하다.

</details>

## 한 줄 정리

> 교차검증으로 안정적으로 튜닝하고, 파이프라인으로 전처리를 train에 묶어 데이터 누수 없이 정직한 성능을 얻는다.
