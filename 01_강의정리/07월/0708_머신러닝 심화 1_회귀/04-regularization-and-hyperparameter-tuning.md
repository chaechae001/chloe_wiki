# 규제와 하이퍼파라미터 튜닝 — 과적합을 억제하고 최적 설정을 찾기

> 계수가 제멋대로 커지면 모델은 과적합됩니다. 규제는 계수에 벌점을 매겨 이를 억제하고, 튜닝은 그 벌점의 세기 같은 설정값을 자동으로 탐색합니다.

`규제` · `Ridge` · `Lasso` · `ElasticNet` · `GridSearch` · `RandomSearch`

## 핵심요약

- 규제(Regularization)는 손실에 "계수 크기에 대한 벌점"을 더해, 계수가 과하게 커지는 것을 막아 과적합을 억제한다.
- L2 규제(Ridge)는 계수의 제곱합에, L1 규제(Lasso)는 계수의 절대값합에 벌점을 준다.
- Lasso는 불필요한 변수의 계수를 정확히 0으로 만들어 변수 선택 효과가 있고, Ridge는 0에 가깝게 줄이되 0으로는 잘 안 만든다.
- ElasticNet은 L1과 L2를 섞은 규제로, 비율(l1_ratio)로 둘의 균형을 조절한다.
- 규제 강도(alpha) 같은 하이퍼파라미터는 GridSearch(격자 전체 탐색)나 RandomSearch(무작위 탐색)로 찾는다.

## 1. 규제 — Ridge·Lasso·ElasticNet

### 1) 정의

규제는 모델이 학습할 때 최소화하는 손실에 **계수의 크기에 대한 벌점 항**을 추가하는 기법입니다. 모델은 이제 "오차를 줄이면서 동시에 계수도 너무 키우지 말라"는 두 가지 목표를 균형 있게 맞춰야 합니다.

```text
L2 규제 (Ridge)      : Loss + α × Σ βⱼ²        계수의 제곱합에 벌점
L1 규제 (Lasso)      : Loss + α × Σ |βⱼ|       계수의 절대값합에 벌점
ElasticNet           : Loss + α × ( γ Σ|βⱼ| + (1−γ) Σ βⱼ² )   L1·L2 혼합

  α (alpha)   : 규제의 세기. 클수록 계수를 더 강하게 억제
  γ (l1_ratio): L1이 차지하는 비율. 1이면 순수 Lasso, 0이면 순수 Ridge
```

### 2) 왜 필요한가

변수가 많거나 서로 상관이 높으면, 회귀 계수가 극단적으로 커지거나 불안정해지면서 과적합이 생깁니다. 규제는 계수에 "성장 억제 벌점"을 매겨 이 폭주를 막습니다. 특히 변수가 표본보다 많은 고차원 상황에서 규제는 사실상 필수입니다.

### 3) 핵심 흐름 재구성

L1과 L2의 결정적 차이는 **계수를 0으로 만드느냐**입니다. Lasso(L1)는 벌점 구조상 덜 중요한 변수의 계수를 정확히 0으로 눌러 버려, 자동으로 변수를 골라 줍니다(희소성, sparsity). Ridge(L2)는 계수를 0 근처로 골고루 줄이지만 완전히 0으로는 잘 만들지 않아, 모든 변수를 조금씩 남깁니다. ElasticNet은 둘을 섞어, "변수 선택도 하면서 상관 높은 변수들을 함께 다루는" 절충안을 제공합니다.

규제 강도 α를 키우면 계수가 더 강하게 눌려 모델이 단순해지고(과소적합 위험), 줄이면 규제가 약해져 과적합 위험이 커집니다. 그래서 α는 튜닝으로 찾아야 하는 대표적 하이퍼파라미터입니다.

### 4) 쉬운 예시

예산 회의에 비유하면, 규제는 "각 부서가 요구하는 예산(계수)에 벌점을 매기는 재무팀"입니다. L2(Ridge)는 모든 부서 예산을 조금씩 삭감하고, L1(Lasso)은 효과가 미미한 부서 예산을 아예 0으로 잘라 버립니다. α는 재무팀이 얼마나 깐깐한지(삭감 강도)를 정하는 손잡이입니다.

### 5) 코드 예시

규제 강도에 따라 계수가 어떻게 변하는지 감을 잡기 위한 최소 예제입니다.

```python
import numpy as np
from sklearn.linear_model import Ridge, Lasso

# 작은 예시 데이터 (변수 3개)
X = np.array([[1, 2, 3], [2, 1, 0], [3, 3, 1], [0, 1, 2], [2, 2, 2]], dtype=float)
y = np.array([6, 3, 9, 3, 7], dtype=float)

for a in [0.1, 10.0]:
    ridge = Ridge(alpha=a).fit(X, y)
    lasso = Lasso(alpha=a).fit(X, y)
    print(f'alpha={a}')
    print(f'  Ridge 계수: {np.round(ridge.coef_, 3)}')
    print(f'  Lasso 계수: {np.round(lasso.coef_, 3)}')
```

Lasso는 α가 커질수록 일부 계수가 정확히 0이 되어 사라지고, Ridge는 계수들이 0 쪽으로 함께 작아지되 대체로 0이 되지는 않는 패턴을 확인할 수 있습니다.

### 6) 헷갈리는 점

α와 λ(람다)는 같은 것을 가리키는 서로 다른 표기입니다. 라이브러리마다 이름이 다를 뿐 "규제 세기"라는 의미는 동일합니다. 또 규제 전에는 반드시 스케일링을 해야 합니다. 변수 스케일이 제각각이면 벌점이 특정 변수에만 불공평하게 적용됩니다.

### 7) 한 줄 정리

> 규제는 계수에 벌점을 매겨 과적합을 억제하며, L1은 변수를 잘라 내고 L2는 골고루 줄이고 ElasticNet은 둘을 섞는다.

## 2. 하이퍼파라미터 튜닝 — Grid vs Random Search

### 1) 정의

하이퍼파라미터는 모델이 학습으로 배우는 값(계수)이 아니라, **사람이 학습 전에 정해 주는 설정값**입니다. 규제 강도 α, 트리 깊이, 이웃 수 등이 여기 속합니다. 튜닝은 이 설정값의 최적 조합을 찾는 과정입니다. GridSearch는 정해 둔 후보를 격자처럼 전부 시도하고, RandomSearch는 정해진 범위에서 무작위로 몇 개만 뽑아 시도합니다.

### 2) 왜 필요한가

같은 알고리즘도 하이퍼파라미터에 따라 성능이 크게 갈립니다. 손으로 몇 개 찍어 보는 것보다, 체계적으로 탐색하고 교차검증으로 검증하면 더 안정적인 최적값을 찾을 수 있습니다.

### 3) 핵심 흐름 재구성

GridSearch는 후보가 적고 이산적일 때 빠짐없이 탐색한다는 장점이 있지만, 후보 개수가 파라미터마다 곱해져(조합 폭발) 파라미터가 많아지면 급격히 느려집니다. RandomSearch는 넓은 연속 범위에서 무작위로 뽑으므로, **같은 시도 횟수로 더 다양한 값을 커버**할 수 있어 파라미터가 많을 때 효율적입니다. 연구적으로도 파라미터가 많을 때 랜덤 탐색이 격자 탐색보다 효율적이라는 결과가 잘 알려져 있습니다.

### 4) 쉬운 예시

넓은 밭에서 보물을 찾는다고 합시다. GridSearch는 밭을 격자로 나눠 모든 칸을 순서대로 파는 방식이고, RandomSearch는 무작위 지점을 정해진 횟수만큼 파는 방식입니다. 보물이 특정 축(파라미터)에만 민감하게 걸려 있다면, 무작위로 흩뿌리는 편이 같은 삽질 횟수로 그 축의 다양한 값을 더 잘 훑습니다.

### 5) 코드 예시

공개 데이터 `diamonds`(표본 3000개)로 Ridge의 α를 GridSearch와 RandomSearch로 각각 15회 탐색해 비교합니다.

```python
import numpy as np, seaborn as sns, time
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split, GridSearchCV, RandomizedSearchCV
from scipy.stats import loguniform

dia = sns.load_dataset('diamonds').sample(3000, random_state=42).copy()
dia['volume'] = dia['x'] * dia['y'] * dia['z']
X = dia[['carat', 'depth', 'table', 'volume']].values
y = dia['price'].values
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.3, random_state=42)
pipe = Pipeline([('s', StandardScaler()), ('m', Ridge())])

# Grid: 로그 스케일 15개 격자
t0 = time.perf_counter()
grid = GridSearchCV(pipe, {'m__alpha': np.logspace(-3, 3, 15)}, cv=3,
                    scoring='neg_root_mean_squared_error', n_jobs=-1).fit(Xtr, ytr)
tg = time.perf_counter() - t0

# Random: 같은 범위에서 무작위 15개
t0 = time.perf_counter()
rand = RandomizedSearchCV(pipe, {'m__alpha': loguniform(1e-3, 1e3)}, n_iter=15, cv=3,
                          scoring='neg_root_mean_squared_error', n_jobs=-1,
                          random_state=42).fit(Xtr, ytr)
tr = time.perf_counter() - t0

print(f'Grid  : alpha={grid.best_params_["m__alpha"]:.4f}, CV RMSE={-grid.best_score_:.2f}, {tg:.3f}s')
print(f'Random: alpha={rand.best_params_["m__alpha"]:.4f}, CV RMSE={-rand.best_score_:.2f}, {tr:.3f}s')
```

```text
Grid  : alpha=1.0000, CV RMSE=1518.50, 0.101s
Random: alpha=3.9080, CV RMSE=1520.48, 0.103s
```

### 코드 목적

같은 탐색 횟수(15회)에서 두 방식이 찾아낸 최적 α와 성능·시간을 비교합니다.

### 코드 흐름

1. 스케일러+Ridge 파이프라인을 만든다(규제 전 스케일링 필수).
2. Grid는 로그 스케일로 균등한 15개 격자점을 전부 시도한다.
3. Random은 같은 범위에서 무작위로 15개를 뽑아 시도한다.
4. 각 방식의 최적 α, 교차검증 RMSE, 소요 시간을 비교한다.

### 실행 결과 해석

파라미터가 α 하나뿐인 이 단순한 문제에서는 두 방식의 성능이 거의 같습니다(CV RMSE 1518.50 vs 1520.48). Grid가 미세하게 나은데, 이는 격자에 최적값 근처인 α=1이 마침 포함되어 있었기 때문입니다. 이렇게 **파라미터가 적을 때는 Grid가 유리**합니다. 반대로 ElasticNet처럼 α와 l1_ratio 두 개를 동시에 튜닝하거나 파라미터가 더 많아지면, 조합이 곱으로 늘어나는 Grid보다 Random이 같은 예산으로 더 넓게 탐색해 유리해집니다.

### 실무 연결

튜닝 예산(시도 횟수·시간)은 항상 유한합니다. 파라미터가 1~2개면 Grid로 촘촘히, 3개 이상이면 Random으로 넓게 훑고 유망 구간을 좁혀 다시 탐색하는 전략이 실무 표준입니다. 튜닝은 반드시 시간까지 함께 측정해 비용 대비 효과를 기록해 둡니다.

### 6) 헷갈리는 점

튜닝은 반드시 **교차검증 위에서** 해야 합니다. 단일 검증 세트에만 맞춰 튜닝하면 그 세트에 과적합됩니다. 또 최종 성능 보고는 튜닝에 쓰지 않은 별도의 test 세트에서 해야 정직합니다.

### 7) 한 줄 정리

> 하이퍼파라미터 튜닝은 최적 설정을 체계적으로 찾는 과정이며, 파라미터가 적으면 Grid, 많으면 Random이 효율적이다.

## 코드로 보기 — 규제·튜닝 파이프라인 흐름

```text
[원본 X, y]
   └─> train / test 분리
         └─> Pipeline( StandardScaler → Ridge/Lasso/ElasticNet )
               └─> GridSearchCV / RandomizedSearchCV (교차검증으로 alpha 탐색)
                     └─> best 모델을 test 세트에서 최종 평가
```

### 코드 목적

규제·튜닝·검증이 어떻게 하나의 안전한 흐름으로 연결되는지 구조로 보여줍니다.

### 실행 결과 해석

스케일링→규제 모델을 파이프라인으로 묶고, 그 파이프라인 전체를 탐색기가 교차검증하므로 누수 없이 α를 찾을 수 있습니다. 최종 숫자는 탐색에 쓰지 않은 test에서 뽑아야 신뢰할 수 있습니다.

### 실무 연결

이 구조를 그대로 여러 알고리즘으로 확장하면 [5편의 모델 비교·앙상블](05-ensemble-stacking-and-shap.md)로 이어집니다.

## 직접 해보기

1. Lasso와 Ridge의 계수 처리 방식 차이를 한 문장으로 설명하세요.
2. 규제 강도 α를 아주 크게 키우면 모델은 어떤 방향(과적합/과소적합)으로 가나요?
3. 튜닝할 파라미터가 4개이고 시간이 제한적일 때 Grid와 Random 중 무엇을 택하는 게 합리적인가요?

<details>
<summary>정답 보기</summary>

1. Lasso(L1)는 덜 중요한 변수의 계수를 정확히 0으로 만들어 변수를 선택하고, Ridge(L2)는 계수를 0 근처로 골고루 줄이되 대체로 0으로 만들지는 않습니다.
2. 과소적합 방향입니다. 계수가 강하게 눌려 모델이 지나치게 단순해지고, 데이터의 패턴조차 제대로 못 잡게 됩니다.
3. RandomSearch입니다. 파라미터가 많으면 Grid는 조합이 곱으로 폭증해 비효율적이므로, 같은 예산으로 더 넓게 훑는 Random이 유리합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| L1(Lasso) vs L2(Ridge) | 절대값 벌점으로 계수를 0으로 vs 제곱 벌점으로 0 근처로 |
| 계수 vs 하이퍼파라미터 | 학습으로 배우는 값 vs 사람이 학습 전에 정하는 설정값 |
| Grid vs Random Search | 격자 전체 탐색 vs 정해진 범위 무작위 탐색 |
| alpha 크게 vs 작게 | 크면 강한 규제(과소적합), 작으면 약한 규제(과적합) |

## 연결되는 개념

- 이전에 알면 좋은 개념: [데이터 누수와 파이프라인·교차검증](03-data-leakage-and-pipeline.md)
- 다음에 이어지는 개념: [모델 비교·스태킹·SHAP](05-ensemble-stacking-and-shap.md)
- 함께 보면 좋은 키워드: `과적합`, `교차검증`, `l1_ratio`

## 셀프 체크

- [ ] 규제가 과적합을 억제하는 원리를 설명할 수 있다.
- [ ] L1·L2·ElasticNet의 차이를 안다.
- [ ] 하이퍼파라미터와 계수의 차이를 구분한다.
- [ ] Grid와 Random Search를 언제 쓰는지 판단할 수 있다.
- [ ] 튜닝은 교차검증 위에서 해야 함을 이해한다.

### 복습 질문 및 답변

**Q1. 규제는 어떻게 과적합을 막나요?**

<details>
<summary>답</summary>

손실에 계수 크기에 대한 벌점 항을 더해, 모델이 오차를 줄이면서도 계수를 과하게 키우지 못하게 합니다. 계수가 눌리면 모델이 단순해져 훈련 데이터의 노이즈에 덜 휘둘립니다.

</details>

**Q2. Lasso가 변수 선택 효과를 갖는 이유는 무엇인가요?**

<details>
<summary>답</summary>

L1 벌점(절대값합)의 기하학적 구조상 최적해에서 일부 계수가 정확히 0이 되는 지점에 잘 걸립니다. 계수가 0이 된 변수는 예측에서 완전히 빠지므로, 결과적으로 중요한 변수만 남기는 선택 효과가 생깁니다.

</details>

**Q3. 파라미터가 하나뿐인데 Grid가 Random보다 잘 나왔습니다. 일반화해도 되나요?**

<details>
<summary>답</summary>

안 됩니다. 파라미터가 적을 때는 Grid가 격자에 최적점을 포함하기 쉬워 유리할 수 있지만, 파라미터가 늘면 조합이 폭증해 Random이 더 효율적입니다. 파라미터 개수와 탐색 예산에 따라 방식을 골라야 합니다.

</details>

## 한 줄 정리

> 규제로 과적합을 억제하고, 그 규제 강도를 교차검증 기반 탐색으로 찾는 것이 견고한 회귀 모델의 핵심 절차다.
