# 의사결정나무와 특성 중요도 — 어떤 질문으로 나눌 것인가

> 의사결정나무는 스무고개와 같습니다. "좋은 질문"이란 한 번에 데이터를 잘 갈라, 섞임(불순도)을 크게 줄이는 질문입니다.

`의사결정나무` · `Gini` · `Entropy` · `정보이득` · `특성중요도`

## 핵심요약

- 의사결정나무는 "이 조건이 참인가?"라는 질문을 반복해 데이터를 나누는 분류·회귀 모델이다.
- 노드가 얼마나 섞여 있는지는 불순도(impurity)로 재며, Gini와 Entropy 두 방법이 있다.
- 좋은 분할은 부모보다 자식 노드의 불순도를 많이 줄이는 것, 즉 정보 이득(Gain)이 큰 분할이다.
- `max_depth`가 너무 작으면 과소적합, 너무 크면 과적합이 되며 train/test 정확도 차이로 확인한다.
- 트리의 feature importance는 "그 특성으로 나눴을 때 줄인 불순도의 누적"이며, 모델마다 중요도의 의미가 다르다.

## 1. 불순도 — Gini와 Entropy

### 1) 정의

불순도(impurity)는 한 노드 안에 여러 클래스가 얼마나 섞여 있는지를 나타내는 값입니다. 한 클래스만 있으면 0, 반반으로 섞이면 최대가 됩니다. 두 가지 계산법이 있습니다.

```
Gini    = 1 − Σ (p_k)²
Entropy = − Σ p_k × log₂(p_k)
```

여기서 p_k는 노드 안에서 클래스 k가 차지하는 비율입니다.

### 2) 왜 필요한가

의사결정나무는 "어떤 질문으로 나눌까?"를 결정할 때, 나눈 뒤 자식 노드가 얼마나 덜 섞이는지를 기준으로 삼습니다. 그 "섞임"을 수치화하는 것이 불순도입니다.

### 3) 쉬운 예시

과일 바구니를 생각해 보세요. 사과만 든 바구니는 순수(불순도 0), 사과와 배가 반반이면 가장 섞인 상태(불순도 최대)입니다. 좋은 질문("빨간색인가?")은 바구니를 사과 쪽과 배 쪽으로 깔끔히 갈라 섞임을 줄입니다.

### 4) 코드 예시 — 불순도 직접 계산

```python
import numpy as np

def gini(y):
    p = np.bincount(y) / len(y)
    return 1 - np.sum(p ** 2)

def entropy(y):
    p = np.bincount(y) / len(y)
    p = p[p > 0]                     # log(0) 방지
    return -np.sum(p * np.log2(p))

for label, arr in [("순수 [0,0,0,0]", [0,0,0,0]),
                   ("조금섞임 [0,0,0,1]", [0,0,0,1]),
                   ("반반 [0,0,1,1]", [0,0,1,1])]:
    a = np.array(arr)
    print(f"{label:20s} Gini={gini(a):.3f}  Entropy={entropy(a):.3f}")
```

실행 결과:

```
순수 [0,0,0,0]         Gini=0.000  Entropy=0.000
조금섞임 [0,0,0,1]       Gini=0.375  Entropy=0.811
반반 [0,0,1,1]         Gini=0.500  Entropy=1.000
```

### 5) 결과 해석

- 한 클래스만 있으면 Gini와 Entropy 모두 0(완전 순수).
- 반반으로 섞이면 Gini는 0.5, Entropy는 1.0으로 최대.
- 두 지표는 스케일(Y축 값)은 다르지만, 어느 지점이 더 섞였는지에 대한 경향은 일치합니다.

### 6) 헷갈리는 점

Gini와 Entropy는 스케일이 달라 값 자체를 비교하면 안 됩니다. 같은 데이터에서 최적 분할 지점의 순위는 대체로 같으므로, 둘 중 무엇을 쓰든 결과는 비슷합니다.

### 7) 한 줄 정리

> 불순도는 노드의 섞임 정도를 숫자로 재는 값이며, Gini와 Entropy는 계산법만 다를 뿐 같은 목적을 가진다.

## 2. 분할 조건 — 정보 이득(Gain)

### 1) 정의

정보 이득(Information Gain)은 부모 노드의 불순도에서, 자식 노드들의 가중 평균 불순도를 뺀 값입니다.

```
Gain = I(parent) − (n_L / n) × I(left) − (n_R / n) × I(right)
```

여기서 n_L, n_R은 왼쪽·오른쪽 자식의 데이터 수, n은 전체 수입니다.

### 2) 왜 필요한가

트리는 가능한 모든 분할 후보(예: `total_bill ≤ 20.5`인가?)를 시험해 보고, Gain이 가장 큰 질문을 골라 노드를 나눕니다. Gain이 클수록 좋은 질문입니다.

### 3) 코드로 보기 — 최적 분할 지점 찾기 (seaborn tips)

`tips` 데이터에서 팁 비율이 중앙값 이상인지(`high_tip`)를 타깃으로, 식사 금액(`total_bill`)의 여러 임곗값마다 Gain을 계산해 최적 분할을 찾습니다.

```python
import seaborn as sns

tips = sns.load_dataset('tips')
tips['tip_rate'] = tips['tip'] / tips['total_bill']
tips['high_tip'] = (tips['tip_rate'] >= tips['tip_rate'].median()).astype(int)

# total_bill 임곗값 후보마다 (부모 불순도 − 자식 가중 불순도) 계산 → Gain 최대 지점 선택
# (전체 계산 로직은 위 gini 함수를 재사용해 좌/우 그룹에 적용)
```

강의 실습에서 Gini·Entropy 두 기준 모두 `total_bill = 22.58` 부근이 Gain을 가장 크게 만드는 최적 분할로 선택되었습니다. 이 지점에서 나누면 왼쪽(22.58 이하) 그룹의 high_tip 비율은 약 58%, 오른쪽(22.58 초과) 그룹은 약 30%로, 두 그룹이 뚜렷하게 갈라집니다.

### 4) 한 줄 정리

> 좋은 분할은 정보 이득이 큰 질문이며, 트리는 모든 후보를 시험해 Gain이 최대인 지점을 고른다.

## 3. max_depth와 과소/과적합

### 1) 정의

`max_depth`는 나무가 질문을 몇 단계까지 이어갈지 정하는 값입니다.

### 2) 왜 필요한가

- 너무 작으면 단순해서 중요한 패턴을 놓칩니다(과소적합).
- 너무 크면 학습 데이터를 통째로 외워 버립니다(과적합).

### 3) 코드 예시 — 깊이별 정확도

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split

cols = ['total_bill', 'size', 'smoker_yes', 'dinner', 'weekend']
# (smoker_yes, dinner, weekend는 tips에서 파생한 0/1 변수)
X = tips[cols]; y = tips['high_tip']
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)

for d in [1, 2, 3, 4, 5, None]:
    t = DecisionTreeClassifier(max_depth=d, random_state=42).fit(X_tr, y_tr)
    print(f"max_depth={str(d):>4}  train={t.score(X_tr, y_tr):.3f}  test={t.score(X_te, y_te):.3f}")
```

실행 결과:

```
max_depth=   1  train=0.612  test=0.527
max_depth=   2  train=0.612  test=0.527
max_depth=   3  train=0.653  test=0.486
max_depth=   4  train=0.653  test=0.486
max_depth=   5  train=0.659  test=0.608
max_depth=None  train=0.994  test=0.595
```

### 4) 결과 해석

- `max_depth=None`(끝까지 분할)에서 train 정확도가 0.994로 치솟지만 test는 0.595에 그칩니다. train과 test의 큰 격차가 전형적인 **과적합** 신호입니다.
- 반대로 깊이 1~2에서는 train·test 모두 낮아 **과소적합** 경향을 보입니다.
- 즉 정확도는 test 기준으로 봐야 하며, train만 높은 모델은 실전에서 무너집니다. 이 데이터에서는 중간 깊이가 상대적으로 나은 균형점입니다.

### 5) 한 줄 정리

> max_depth가 작으면 과소적합, 너무 크면 과적합이며 train/test 격차로 과적합을 감지한다.

## 4. Tree Feature Importance

### 1) 정의

트리의 `feature_importances_`는 각 특성이 만든 불순도 감소량을 누적한 값입니다.

```
Importance_j = Σ (n_t / N) × ΔI_t     (특성 j로 분할한 노드 t에 대해 합산)
```

즉, 특성이 몇 번 쓰였는지가 아니라 "그 특성으로 나눴을 때 불순도를 얼마나 줄였는지"를 봅니다.

### 2) 코드 예시 — 직접 계산 vs sklearn

```python
tree = DecisionTreeClassifier(max_depth=3, random_state=42).fit(X, y)
import pandas as pd
imp = pd.DataFrame({'feature': cols, 'importance': tree.feature_importances_.round(4)})
print(imp.sort_values('importance', ascending=False).to_string(index=False))
```

실행 결과:

```
   feature  importance
total_bill       0.956
smoker_yes       0.044
      size       0.000
    dinner       0.000
   weekend       0.000
```

### 3) 결과 해석

`total_bill`이 불순도 감소의 대부분을 차지해 압도적으로 중요합니다. 나머지 특성은 이 트리에서 거의 분할에 쓰이지 않아 중요도가 0에 가깝습니다. 중요도 0이라고 해서 "무의미한 변수"라는 뜻은 아니며, 이 트리 구조에서 우선순위가 밀렸을 뿐입니다.

### 4) 한 줄 정리

> 트리 feature importance는 그 특성이 줄인 불순도의 누적이며, 자주 쓰였는지가 아니라 얼마나 잘 갈랐는지를 본다.

## 5. 모델마다 다른 feature importance

feature importance는 모델마다 같은 뜻이 아닙니다.

- **Linear Regression**: 계수(coefficient). 특성이 1 증가할 때 예측값이 얼마나 변하는지.
- **Logistic Regression**: log-odds에 대한 계수. 부호와 크기로 방향·영향력.
- **Decision Tree / RandomForest**: 불순도 감소량.
- **KNN**: 자체 importance가 없어, 보통 permutation importance로 본다.
- **Naive Bayes**: 클래스별 평균 차이를 간이 중요도로 볼 수 있다.
- **Neural Network**: 직접 해석이 어려워 permutation importance나 SHAP 같은 model-agnostic 방법을 쓴다.

같은 데이터라도 모델이 데이터를 보는 방식이 다르므로, 서로 다른 모델의 중요도 숫자를 같은 기준으로 비교하면 안 됩니다.

## 직접 해보기

1. `[0,0,1,1]`과 `[0,0,0,0]`의 Gini를 각각 구하고, 어느 쪽이 좋은 분할 결과인지 설명하라.
2. train 정확도 0.99, test 정확도 0.60인 트리가 있다. 무슨 문제이며 어떻게 완화하는가?
3. 로지스틱 회귀의 "중요도"와 의사결정나무의 "중요도"는 왜 직접 비교하면 안 되는가?

<details>
<summary>정답 보기</summary>

1. `[0,0,1,1]`은 Gini = 1 − (0.5² + 0.5²) = 0.5(최대 섞임), `[0,0,0,0]`은 Gini = 0(완전 순수). 분할 결과로는 `[0,0,0,0]`처럼 불순도가 낮은 쪽이 좋다.
2. 과적합이다. train은 거의 완벽한데 test가 크게 낮아 학습 데이터를 외운 상태다. `max_depth`를 줄이거나 `min_samples_leaf`를 키우는 등 가지치기로 완화한다.
3. 로지스틱 회귀의 중요도는 log-odds 계수(방향·크기)이고, 트리의 중요도는 불순도 감소량이다. 정의와 스케일이 완전히 달라 같은 축에서 비교하는 것은 의미가 없다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| Gini vs Entropy | 계산법·스케일은 다르지만 최적 분할 순위는 대체로 동일 |
| 과소적합 vs 과적합 | 너무 단순해 패턴 놓침 vs 너무 복잡해 학습 데이터 암기 |
| 분할 횟수 vs 불순도 감소 | 특성이 자주 쓰인 것 ≠ 중요, 얼마나 잘 갈랐는지가 중요 |
| 선형모델 계수 vs 트리 중요도 | 기울기(방향·크기) vs 불순도 감소량. 서로 비교 불가 |
| 중요도 0 vs 무의미 | 이 트리에서 안 쓰였을 뿐, 변수 자체가 쓸모없다는 뜻은 아님 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [데이터 누수와 파이프라인](04-data-leakage.md)
- 다음에 이어지는 개념: [모델별 특성 엔지니어링](06-feature-engineering.md)
- 함께 보면 좋은 키워드: `impurity`, `Gain`, `max_depth`, `permutation importance`

## 셀프 체크

- [ ] Gini와 Entropy의 의미와 계산법을 안다.
- [ ] 정보 이득(Gain)으로 좋은 분할을 판단할 수 있다.
- [ ] max_depth와 과소/과적합의 관계를 안다.
- [ ] 트리 feature importance의 정의를 설명할 수 있다.
- [ ] 모델마다 중요도의 의미가 다름을 안다.

### 복습 질문 및 답변

**Q1. 좋은 분할 질문의 조건은?**

<details>
<summary>답</summary>

부모 노드보다 자식 노드의 불순도를 많이 줄이는 질문, 즉 정보 이득(Gain)이 큰 질문이다. 트리는 모든 분할 후보를 시험해 Gain이 최대인 지점을 고른다.

</details>

**Q2. train 정확도만 계속 오르는데 test가 정체하거나 떨어진다. 무엇을 조정하나?**

<details>
<summary>답</summary>

과적합 신호다. `max_depth`를 제한하거나 `min_samples_leaf`를 키워 나무의 복잡도를 낮추는 가지치기로 일반화 성능을 회복한다.

</details>

**Q3. 트리 feature importance가 0인 특성은 버려도 되는가?**

<details>
<summary>답</summary>

단정할 수 없다. 이 트리 구조에서 분할에 쓰이지 않았을 뿐, 다른 모델이나 다른 특성 조합에서는 유용할 수 있다. 중요도 0을 "무가치"로 곧장 해석하면 안 된다.

</details>

## 한 줄 정리

> 의사결정나무는 정보 이득이 큰 질문으로 데이터를 나누고, 그 과정에서 줄인 불순도로 특성 중요도를 매기며, 중요도의 의미는 모델마다 다르다.
