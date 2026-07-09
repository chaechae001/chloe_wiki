# 스태킹 앙상블 — 여러 모델의 예측을 다시 학습하기

> 서로 다른 모델의 예측을 재료 삼아 "심판 모델"이 최종 결정을 내리는 기법.

`스태킹` · `앙상블` · `메타모델` · `베이스모델` · `모델결합`

## 핵심요약

- 스태킹은 여러 베이스 모델의 예측을 입력으로 삼아 메타 모델이 최종 예측을 학습한다.
- 서로 성격이 다른 모델을 섞을수록 결합 효과가 커지는 경향이 있다.
- 메타 모델은 각 베이스 모델을 언제 얼마나 믿을지 학습한다.
- 스태킹이 항상 최고는 아니다. 단일 최고 모델을 못 이길 때도 있다.
- 누수를 막기 위해 베이스 예측은 교차검증 방식으로 만들어야 한다.

---

## 1. 스태킹의 아이디어

### 1) 정의

스태킹(Stacking)은 앙상블의 한 방식으로, 여러 개의 서로 다른 모델(베이스 모델)의 예측 결과를 새로운 입력 특성으로 만들고, 그 위에 또 다른 모델(메타 모델)을 얹어 최종 예측을 학습한다.

### 2) 왜 필요한가

모델마다 잘 맞히는 영역이 다르다. 트리는 비선형 관계에, 로지스틱 회귀는 선형 경향에 강할 수 있다. 스태킹은 각 모델의 강점을 상황별로 조합해, 단일 모델보다 안정적이고 종종 더 나은 성능을 노린다.

### 3) 핵심 흐름 재구성

- 여러 베이스 모델을 학습한다(예: 랜덤 포레스트, 부스팅, KNN).
- 각 베이스 모델의 예측(확률)을 모아 새 특성 행렬을 만든다.
- 메타 모델(보통 단순한 로지스틱 회귀)이 이 예측들을 입력으로 최종 판단을 학습한다.
- 누수 방지를 위해 베이스 예측은 교차검증(out-of-fold) 방식으로 생성한다.

### 4) 쉬운 예시

세 명의 전문가에게 의견을 묻고, 사회자가 "이 전문가는 이런 상황에서 잘 맞더라"를 학습해 최종 결론을 내는 것과 같다. 사회자가 메타 모델, 전문가가 베이스 모델이다.

### 5) 코드 예시

```python
from sklearn.ensemble import (RandomForestClassifier,
                              GradientBoostingClassifier, StackingClassifier)
from sklearn.linear_model import LogisticRegression
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import StratifiedKFold

base = [
    ('rf',  RandomForestClassifier(n_estimators=300, random_state=42)),
    ('gb',  GradientBoostingClassifier(random_state=42)),
    ('knn', KNeighborsClassifier(n_neighbors=15)),
]
cv = StratifiedKFold(5, shuffle=True, random_state=42)

stack = StackingClassifier(
    estimators=base,
    final_estimator=LogisticRegression(max_iter=1000),  # 메타 모델
    cv=cv,      # out-of-fold 예측으로 누수 방지
)
```

`cv` 인자가 베이스 예측을 교차검증 방식으로 만들어 누수를 막는 핵심이다.

### 6) 헷갈리는 점

- 메타 모델은 보통 단순한 것이 좋다. 복잡한 메타 모델은 과적합을 부른다.
- 스태킹은 배깅·부스팅과 다르다. 배깅은 같은 종류 모델의 평균, 부스팅은 순차 보완, 스태킹은 서로 다른 모델의 예측을 재학습하는 것이다.

### 7) 한 줄 정리

> 스태킹은 여러 모델의 예측을 재료로 메타 모델이 최종 결정을 학습하는 결합 기법이다.

---

## 2. 스태킹은 항상 이길까

### 1) 정의

스태킹은 여러 모델을 결합하므로 대개 안정적이지만, 반드시 단일 최고 모델을 이긴다는 보장은 없다.

### 2) 왜 필요한가

"앙상블이면 무조건 좋다"는 오해를 피해야 한다. 결합이 오히려 최고 모델의 강점을 희석할 수도 있으므로, 단일 모델과 항상 비교해 판단해야 한다.

### 3) 핵심 흐름 재구성

- 베이스 모델들이 서로 비슷하면(상관이 높으면) 결합 이득이 작다.
- 베이스 모델이 다양하고 서로 다른 실수를 할수록 결합 효과가 커진다.
- 스태킹은 계산 비용과 복잡도가 크므로, 성능 향상이 그 비용을 정당화하는지 봐야 한다.

### 4) 쉬운 예시

같은 신문만 읽는 세 사람에게 물으면 답이 거의 같아 종합해도 별 이득이 없다. 서로 다른 출처를 보는 사람들을 모아야 종합 판단이 좋아진다. 베이스 모델의 다양성이 이 "출처의 다양성"이다.

### 5) 코드 예시

```python
from sklearn.metrics import accuracy_score, f1_score

# 개별 베이스 모델과 스태킹을 같은 test로 비교
for name, est in base:
    est.fit(X_tr, y_tr)
    print(f"{name:4s} acc={accuracy_score(y_te, est.predict(X_te)):.3f} "
          f"f1={f1_score(y_te, est.predict(X_te)):.3f}")

stack.fit(X_tr, y_tr)
sp = stack.predict(X_te)
print(f"STACK acc={accuracy_score(y_te, sp):.3f} f1={f1_score(y_te, sp):.3f}")
```

### 6) 헷갈리는 점

- 스태킹 성능이 단일 최고 모델과 비슷하다고 실패가 아니다. 여러 시드·데이터에서 더 안정적이면 그 자체로 가치가 있다.
- 베이스 모델을 무작정 많이 넣는다고 좋아지지 않는다. 다양성과 품질이 개수보다 중요하다.

### 7) 한 줄 정리

> 스태킹의 이득은 베이스 모델의 다양성에서 나오며, 단일 최고 모델과 항상 비교해야 한다.

---

## 코드로 보기 — 개별 모델 vs 스태킹 (파이프라인 결합)

```python
import seaborn as sns
from sklearn.model_selection import train_test_split, StratifiedKFold
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import (RandomForestClassifier,
                              GradientBoostingClassifier, StackingClassifier)
from sklearn.linear_model import LogisticRegression
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score

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
def mk(model):
    return Pipeline([('pre', pre), ('model', model)])   # 각 모델에 전처리 결합

cv = StratifiedKFold(5, shuffle=True, random_state=42)
base = [
    ('rf',  mk(RandomForestClassifier(n_estimators=300, random_state=42))),
    ('gb',  mk(GradientBoostingClassifier(random_state=42))),
    ('knn', mk(KNeighborsClassifier(n_neighbors=15))),
]

for name, est in base:
    est.fit(X_tr, y_tr)
    print(f"{name:4s} acc={accuracy_score(y_te, est.predict(X_te)):.3f} "
          f"f1={f1_score(y_te, est.predict(X_te)):.3f}")

stack = StackingClassifier(estimators=base,
                           final_estimator=LogisticRegression(max_iter=1000),
                           cv=cv, n_jobs=-1)
stack.fit(X_tr, y_tr)
sp = stack.predict(X_te)
print(f"STACK acc={accuracy_score(y_te, sp):.3f} "
      f"f1={f1_score(y_te, sp):.3f} "
      f"auc={roc_auc_score(y_te, stack.predict_proba(X_te)[:, 1]):.3f}")
```

### 코드 목적

전처리를 각 베이스 모델에 결합한 뒤 스태킹으로 묶어, 개별 모델과 스태킹의 성능을 정직하게 비교하기 위한 코드다.

### 코드 흐름

1. 수치형·범주형 전처리를 정의한다.
2. 각 베이스 모델을 전처리와 결합한 파이프라인으로 만든다.
3. 개별 모델의 test 성능을 측정한다.
4. 스태킹으로 묶어 최종 성능을 비교한다.

### 실행 결과 해석

```text
rf   acc=0.816 f1=0.744
gb   acc=0.793 f1=0.704
knn  acc=0.804 f1=0.724
STACK acc=0.804 f1=0.720 auc=0.845
```

이 데이터에서는 스태킹(0.804)이 단일 랜덤 포레스트(0.816)를 넘지 못했다. 베이스 모델들의 실수가 비슷했거나 결합 이득이 크지 않았다는 뜻이다. 이는 "스태킹이 항상 최고는 아니다"를 잘 보여주는 결과이며, 실제로 이런 경우 단일 모델을 배포하는 편이 더 단순하고 합리적일 수 있다.

### 실무 연결

캐글 같은 경진대회에서는 마지막 성능 한 방울을 짜내기 위해 스태킹을 자주 쓴다. 반면 실서비스에서는 복잡도·지연·유지보수 비용 때문에 성능 향상이 확실할 때만 채택한다. "성능 이득 대비 복잡도"를 저울질하는 것이 핵심이다.

---

## 직접 해보기

1. 스태킹에서 베이스 모델과 메타 모델의 역할을 각각 설명하라.
2. 베이스 모델을 고를 때 왜 서로 다른 종류를 섞는 것이 유리한가?
3. 스태킹 성능이 단일 최고 모델과 거의 같다면, 실서비스에서 무엇을 선택하는 게 합리적일 수 있는가?

<details>
<summary>정답 보기</summary>

1. 베이스 모델은 각자 예측을 내놓는 1차 모델이고, 메타 모델은 그 예측들을 입력으로 받아 언제 어떤 모델을 믿을지 학습해 최종 예측을 내는 2차 모델이다.
2. 서로 다른 모델은 서로 다른 실수를 하는 경향이 있어, 결합하면 약점이 상쇄되고 강점이 보완된다. 비슷한 모델만 모으면 실수도 비슷해 결합 이득이 작다.
3. 단일 모델을 선택하는 것이 합리적일 수 있다. 스태킹은 복잡도·계산비용·유지보수 부담이 크므로, 성능 이득이 뚜렷하지 않으면 단순한 단일 모델이 운영에 유리하다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 스태킹 vs 배깅 | 서로 다른 모델 예측을 재학습하면 스태킹, 같은 모델을 병렬 평균하면 배깅 |
| 스태킹 vs 부스팅 | 예측을 메타 모델로 결합하면 스태킹, 오차를 순차 보완하면 부스팅 |
| 베이스 모델 vs 메타 모델 | 1차 예측을 내면 베이스, 그 예측을 종합하면 메타 |
| 다양성 vs 개수 | 서로 다른 실수를 하는 다양성이 단순한 모델 개수보다 중요 |

## 연결되는 개념

- 이전에 알면 좋은 개념: 하이퍼파라미터 튜닝·교차검증·파이프라인
- 다음에 이어지는 개념: 모델 해석(SHAP), 시계열 예측 등 심화 주제
- 함께 보면 좋은 키워드: `out-of-fold`, `메타학습`, `모델다양성`

## 셀프 체크

- [ ] 스태킹의 2단계 구조를 설명할 수 있다.
- [ ] 베이스 모델 다양성의 중요성을 안다.
- [ ] 메타 모델을 단순하게 두는 이유를 안다.
- [ ] 스태킹이 항상 최고가 아님을 예로 들 수 있다.
- [ ] 스태킹의 누수 방지 방법(out-of-fold)을 안다.

### 복습 질문 및 답변

**Q1. 스태킹에서 베이스 예측을 교차검증 방식으로 만드는 이유는?**

<details>
<summary>답</summary>

베이스 모델이 학습에 쓴 데이터로 예측한 값을 메타 모델에 넣으면 정답을 이미 본 예측이라 누수가 생긴다. out-of-fold 예측(각 폴드에서 학습에 안 쓰인 조각으로 예측)을 쓰면 메타 모델이 정직한 예측만 학습해 누수를 막는다.

</details>

**Q2. 메타 모델로 복잡한 모델을 쓰면 어떤 위험이 있나?**

<details>
<summary>답</summary>

베이스 예측 위에서 다시 과적합될 위험이 크다. 메타 모델은 예측들의 가중치를 조정하는 역할이라 로지스틱 회귀처럼 단순한 모델로 충분한 경우가 많다.

</details>

**Q3. 앙상블(스태킹)을 도입하기 전 반드시 확인할 것은?**

<details>
<summary>답</summary>

단일 최고 모델과의 성능 비교다. 결합이 실제로 유의미하게 나은지, 그 이득이 복잡도·비용 증가를 정당화하는지를 확인한 뒤 도입 여부를 정해야 한다.

</details>

## 한 줄 정리

> 스태킹은 다양한 모델의 예측을 메타 모델로 결합하는 기법이며, 누수를 막고 단일 모델과 비교해 이득이 확실할 때 가치가 있다.
