# 모델 비교·스태킹·SHAP — 여러 모델을 견주고, 합치고, 설명하기

> 하나의 알고리즘만 믿기보다, 여러 모델을 같은 조건에서 겨루게 하고 그중 강한 것들을 합칩니다. 그리고 SHAP으로 "왜 그렇게 예측했는가"를 열어 봅니다.

`모델비교` · `리더보드` · `Stacking` · `앙상블` · `SHAP`

## 핵심요약

- 여러 알고리즘을 같은 전처리·같은 교차검증 조건에서 비교해 리더보드로 정리하면 공정한 성능 순위를 얻는다.
- Stacking(스태킹)은 서로 다른 여러 모델의 예측을 다시 하나의 메타 모델에 입력해 합치는 앙상블 기법이다.
- 스태킹이 항상 단일 최고 모델을 이기는 것은 아니며, 기반 모델들이 서로 다를수록(다양성) 효과가 커진다.
- SHAP은 각 변수가 개별 예측을 얼마나 밀어 올리거나 내렸는지를 수치로 분해해 모델을 설명한다.
- summary plot은 전체 변수 중요도와 방향을, waterfall plot은 예측 한 건의 분해를 보여준다.

## 1. 모델 비교와 스태킹

### 1) 정의

모델 비교는 여러 알고리즘을 **동일한 조건**(같은 데이터 분할, 같은 전처리, 같은 교차검증)에서 학습해 성능을 나란히 세우는 것입니다. 스태킹(Stacking)은 여러 모델의 예측값들을 새로운 입력으로 삼아, 그 위에서 최종 예측을 내는 메타 모델(보통 단순한 선형 모델)을 한 층 더 얹는 앙상블입니다.

### 2) 왜 필요한가

어떤 알고리즘이 최고인지는 데이터마다 다릅니다. 미리 정답을 알 수 없으니 여러 개를 공정하게 겨루게 해야 합니다. 또 모델마다 잘 맞히는 영역이 다르기 때문에, 이들을 잘 합치면 단일 모델보다 더 안정적인 예측을 얻을 수 있습니다. 스태킹은 이 "합치기"를 학습으로 최적화합니다.

### 3) 핵심 흐름 재구성

스태킹의 구조는 2단계입니다.

```text
1단계 (기반 모델, base): 서로 다른 여러 모델이 각자 예측을 낸다.
2단계 (메타 모델, meta): 그 예측들을 입력으로 받아 최종 예측을 학습한다.

  핵심: 기반 모델들이 서로 다른 방식으로 틀려야(다양성) 합쳤을 때 이득이 크다.
        비슷하게 틀리는 모델끼리 합치면 개선이 거의 없다.
```

여기서 중요한 점은, 메타 모델이 기반 모델의 예측을 학습할 때 **누수를 막기 위해 교차검증 방식으로 만든 예측**을 써야 한다는 것입니다. 사이킷런 `StackingRegressor`는 이를 내부적으로 처리해 줍니다.

### 4) 쉬운 예시

의사 여러 명에게 각자 진단을 받은 뒤, 경험 많은 종합의(메타 모델)가 그 진단들을 종합해 최종 판단을 내리는 것과 같습니다. 전공이 서로 다른 의사들일수록 종합의 판단이 풍부해집니다. 모두 같은 전공이면 의견이 겹쳐 종합의 이점이 줄어듭니다.

### 5) 코드 예시

`diamonds`로 여러 모델을 교차검증 비교해 리더보드를 만들고, 상위 3개를 스태킹합니다.

```python
import numpy as np, pandas as pd, seaborn as sns, time
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge, Lasso
from sklearn.neighbors import KNeighborsRegressor
from sklearn.tree import DecisionTreeRegressor
from sklearn.ensemble import (RandomForestRegressor, GradientBoostingRegressor,
                              StackingRegressor)
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import mean_squared_error, r2_score

dia = sns.load_dataset('diamonds').sample(4000, random_state=42).copy()
dia['volume'] = dia['x'] * dia['y'] * dia['z']
X = dia[['carat', 'depth', 'table', 'volume']].values
y = dia['price'].values
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, random_state=42)

models = {
    'Ridge': Ridge(alpha=1.0), 'Lasso': Lasso(alpha=1.0),
    'KNN': KNeighborsRegressor(n_neighbors=7),
    'DecisionTree': DecisionTreeRegressor(max_depth=6, random_state=42),
    'RandomForest': RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1),
    'GradientBoosting': GradientBoostingRegressor(random_state=42),
}
rows = []
for name, est in models.items():
    pipe = Pipeline([('s', StandardScaler()), ('m', est)])
    cv = -cross_val_score(pipe, Xtr, ytr, cv=3,
                          scoring='neg_root_mean_squared_error', n_jobs=-1)
    rows.append((name, cv.mean(), cv.std()))
lb = pd.DataFrame(rows, columns=['model', 'cv_rmse', 'cv_std']).sort_values('cv_rmse')
print(lb.to_string(index=False))
```

```text
           model      cv_rmse    cv_std
GradientBoosting  1447.463017  45.007...
    RandomForest  1489.019...  35.21...
             KNN  1495.85...   29.75...
           Ridge  1542.63...   18.46...
           Lasso  1543.17...   18.36...
    DecisionTree  1561.72...   78.10...
```

이어서 상위 3개(GradientBoosting·RandomForest·KNN)를 스태킹해 단일 최고 모델과 test에서 비교합니다.

```python
best_name = lb.iloc[0]['model']
best_pipe = Pipeline([('s', StandardScaler()), ('m', models[best_name])]).fit(Xtr, ytr)
rmse_single = np.sqrt(mean_squared_error(yte, best_pipe.predict(Xte)))

top3 = lb['model'].head(3).tolist()
estimators = [(n, Pipeline([('s', StandardScaler()), ('m', models[n])])) for n in top3]
stack = StackingRegressor(estimators=estimators,
                          final_estimator=Ridge(alpha=1.0), cv=3, n_jobs=-1).fit(Xtr, ytr)
rmse_stack = np.sqrt(mean_squared_error(yte, stack.predict(Xte)))
print(f'Best Single({best_name}): Test RMSE={rmse_single:.2f}')
print(f'Stacking(Top3)         : Test RMSE={rmse_stack:.2f}')
```

```text
Best Single(GradientBoosting): Test RMSE=1250.33
Stacking(Top3)               : Test RMSE=1253.74
```

### 코드 목적

여러 모델을 공정하게 비교해 순위를 내고, 상위 모델을 합친 스태킹이 단일 최고 모델을 실제로 이기는지 검증합니다.

### 코드 흐름

1. 6개 모델을 각각 스케일러와 묶어 동일한 교차검증으로 평가한다.
2. CV RMSE 기준으로 리더보드를 정렬한다.
3. 상위 3개를 기반 모델로, Ridge를 메타 모델로 스태킹한다.
4. 단일 최고 모델과 스태킹을 test 세트에서 비교한다.

### 실행 결과 해석

리더보드에서는 GradientBoosting이 CV RMSE 1447로 1위, RandomForest·KNN이 뒤를 잇습니다. 그런데 test에서 **스태킹(1253.74)이 단일 최고 모델(1250.33)을 이기지 못했습니다.** 차이는 미미하지만 오히려 살짝 나빴습니다. 이유는 상위 3개가 모두 비선형 모델이라 예측 패턴이 서로 비슷했기 때문입니다. 다양성이 부족하면 스태킹의 이점이 사라진다는 점을 실제 숫자로 보여주는 정직한 결과입니다. 스태킹을 쓸 때는 성격이 다른 모델(선형·트리·이웃 기반 등)을 섞는 것이 중요합니다.

### 실무 연결

캐글 대회 상위권은 종종 스태킹으로 마지막 성능을 짜냅니다. 다만 스태킹은 복잡도와 학습 시간이 크게 늘고 항상 이득이 보장되지도 않으므로, 실무에서는 "얻는 성능 대비 운영 비용"을 따져 도입합니다. 단일 모델로 충분하면 단순함이 유지보수에 유리합니다.

### 6) 헷갈리는 점

"앙상블이면 무조건 좋아진다"는 오해가 흔합니다. 기반 모델들이 비슷하게 예측하고 비슷하게 틀리면 합쳐도 개선이 없습니다. 앙상블의 힘은 성능 자체가 아니라 **다양성**에서 나옵니다.

### 7) 한 줄 정리

> 모델 비교로 공정한 순위를 얻고, 성격이 다른 모델들을 스태킹으로 합칠 때 비로소 앙상블의 이점이 살아난다.

## 2. SHAP — 모델의 판단 근거 열어 보기

### 1) 정의

SHAP(SHapley Additive exPlanations)은 게임이론의 섀플리 값을 이용해, **각 변수가 개별 예측값을 기준선에서 얼마나 밀어 올리거나 내렸는지**를 숫자로 분해하는 설명 기법입니다.

### 2) 왜 필요한가

트리 앙상블 같은 강력한 모델은 예측은 잘하지만 내부가 블랙박스라 "왜 이렇게 예측했는지" 설명하기 어렵습니다. 규제 회귀의 계수처럼 단순 해석이 통하지 않죠. SHAP은 어떤 모델에도 적용 가능한 방식으로 이 설명 가능성을 제공합니다. 금융·의료처럼 근거 제시가 의무인 도메인에서 특히 중요합니다.

### 3) 핵심 흐름 재구성

SHAP의 핵심 아이디어는 "예측값 = 기준값(평균 예측) + 각 변수의 기여도 합"입니다.

```text
예측값 = 기준값(base value) + Σ (각 변수의 SHAP 값)

  SHAP 값이 양수  : 그 변수가 예측을 기준선보다 위로 밀어 올림
  SHAP 값이 음수  : 그 변수가 예측을 기준선보다 아래로 끌어내림
```

두 가지 대표 시각화가 있습니다. summary plot은 여러 샘플을 모아 **어떤 변수가 전반적으로 중요하고 어느 방향으로 작용하는지**(글로벌 해석)를 보여주고, waterfall plot은 **한 건의 예측**이 각 변수 기여로 어떻게 조립되는지(로컬 해석)를 단계별로 보여줍니다.

### 4) 쉬운 예시

집값 예측을 SHAP으로 열면, "이 집은 평균보다 비싸게 예측됐는데, 면적이 넓어서 +5000만원, 역세권이라 +3000만원, 대신 오래돼서 −2000만원"처럼 예측값을 항목별 영수증으로 분해해 줍니다. 어떤 요인이 값을 올렸고 내렸는지 한눈에 보입니다.

### 5) 코드 예시

학습된 모델에 SHAP을 적용하는 기본 흐름입니다(개념 이해용 최소 구조).

```python
import shap
import pandas as pd

# best_model: 앞서 학습한 파이프라인, feat_cols: 변수 이름 목록
X_sample = pd.DataFrame(X_tr[:300], columns=feat_cols)     # 속도를 위해 300건만
explainer = shap.Explainer(best_model.predict, X_sample)   # 예측 함수 기반 설명자
shap_values = explainer(X_sample)                          # 각 행·변수의 기여도 계산

shap.summary_plot(shap_values, X_sample)                   # 글로벌: 변수 중요도+방향
shap.plots.waterfall(shap_values[0])                       # 로컬: 1건 예측 분해
```

### 코드 목적

블랙박스 모델의 예측을 변수별 기여도로 분해해, 글로벌·로컬 두 관점에서 설명합니다.

### 코드 흐름

1. 계산 속도를 위해 샘플 일부(300건)만 추린다.
2. 모델의 예측 함수를 기반으로 SHAP 설명자를 만든다.
3. 각 행·각 변수의 기여도(SHAP 값)를 계산한다.
4. summary plot으로 전체 경향을, waterfall plot으로 개별 예측을 본다.

### 실행 결과 해석

summary plot에서는 위쪽에 놓인 변수일수록 예측에 큰 영향을 줍니다. 점의 색(변수 값의 크고 작음)과 좌우 위치(SHAP 값의 부호)를 함께 보면, 예컨대 "무게가 클수록(붉은 점) 가격 예측을 오른쪽(양의 방향)으로 강하게 밀어 올린다"는 관계를 읽을 수 있습니다. waterfall plot에서는 기준값에서 시작해 변수별 막대가 예측값을 위아래로 조립해 가는 과정을 보여, 특정 한 건이 왜 그 값으로 예측됐는지 항목별로 설명됩니다.

### 실무 연결

SHAP은 모델 검수(예상과 다른 방향으로 작용하는 변수 발견), 이해관계자 설명(왜 이 고객이 고위험으로 분류됐는지), 규제 대응(설명 의무)에 두루 쓰입니다. 예측 정확도만큼이나 "설명 가능성"이 중요한 현장에서 표준 도구로 자리 잡았습니다.

### 6) 헷갈리는 점

SHAP 값은 상관이 아니라 **모델의 예측에 대한 기여도**입니다. "변수 X의 SHAP이 크다"는 것은 "그 모델이 X를 예측에 많이 활용한다"는 뜻이지, X가 결과의 진짜 원인이라는 인과 증명이 아닙니다.

### 7) 한 줄 정리

> SHAP은 예측을 기준값과 변수별 기여도의 합으로 분해해, 블랙박스 모델을 글로벌·로컬 양쪽에서 설명한다.

## 코드로 보기 — 비교→앙상블→해석 파이프라인

```text
[여러 모델] → 동일 조건 교차검증 → 리더보드 정렬
      → 상위 & 다양한 모델 선택 → StackingRegressor로 합치기
            → 단일 최고 vs 스태킹 test 비교 → 최종 모델 SHAP 해석
```

### 코드 목적

성능 비교부터 앙상블, 설명까지 하나의 흐름으로 연결해, 모델링의 마지막 단계를 구조화합니다.

### 실행 결과 해석

이번 데이터에서는 스태킹의 이득이 없었지만, 이 흐름 자체는 어떤 회귀 문제에도 재사용할 수 있는 표준 절차입니다. 데이터가 바뀌면 결론(스태킹 채택 여부)도 바뀝니다.

### 실무 연결

이 절차를 템플릿으로 두면, 새로운 회귀 과제를 받았을 때 빠르게 베이스라인부터 최종 해석까지 밟아 나갈 수 있습니다.

## 직접 해보기

1. 스태킹이 단일 최고 모델을 이기지 못하는 경우는 주로 어떤 상황인가요?
2. SHAP의 summary plot과 waterfall plot은 각각 무엇을 설명하나요?
3. 어떤 변수의 SHAP 값이 크게 나왔습니다. 이것을 "그 변수가 결과의 원인"이라고 말해도 될까요?

<details>
<summary>정답 보기</summary>

1. 기반 모델들이 서로 비슷하게 예측하고 비슷하게 틀릴 때(다양성 부족)입니다. 예측 패턴이 겹치면 합쳐도 개선 여지가 없습니다.
2. summary plot은 여러 샘플을 모아 변수의 전반적 중요도와 작용 방향(글로벌)을, waterfall plot은 예측 한 건이 변수별 기여로 어떻게 조립되는지(로컬)를 설명합니다.
3. 안 됩니다. SHAP 값은 "모델이 그 변수를 예측에 얼마나 활용했는가"를 뜻할 뿐, 인과관계의 증명이 아닙니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 단일 모델 vs 스태킹 | 하나의 알고리즘 vs 여러 모델의 예측을 메타 모델로 합침 |
| 앙상블 효과의 근원 | 개별 성능이 아니라 모델 간 다양성에서 나옴 |
| summary plot vs waterfall plot | 전체 변수 중요도(글로벌) vs 예측 1건 분해(로컬) |
| SHAP 기여도 vs 인과관계 | 모델 활용도 설명일 뿐, 진짜 원인 증명은 아님 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [규제와 하이퍼파라미터 튜닝](04-regularization-and-hyperparameter-tuning.md)
- 다음에 이어지는 개념: [시계열 정상성과 ACF·PACF](06-time-series-stationarity-and-acf-pacf.md)
- 함께 보면 좋은 키워드: `GradientBoosting`, `설명가능성`, `메타모델`

## 셀프 체크

- [ ] 여러 모델을 공정하게 비교하는 조건을 설명할 수 있다.
- [ ] 스태킹의 2단계 구조를 이해한다.
- [ ] 앙상블 효과가 다양성에서 나옴을 안다.
- [ ] SHAP의 기본 아이디어(기준값+기여도 합)를 설명할 수 있다.
- [ ] summary와 waterfall plot의 용도를 구분한다.

### 복습 질문 및 답변

**Q1. 여러 모델을 비교할 때 "동일 조건"이 왜 중요한가요?**

<details>
<summary>답</summary>

전처리나 데이터 분할이 모델마다 다르면, 성능 차이가 알고리즘 때문인지 조건 때문인지 구분할 수 없습니다. 같은 파이프라인·같은 교차검증으로 맞춰야 순위가 공정합니다.

</details>

**Q2. 스태킹의 메타 모델은 무엇을 입력으로 받나요?**

<details>
<summary>답</summary>

1단계 기반 모델들이 낸 예측값들을 입력으로 받아, 그것들을 어떻게 조합할지 학습합니다. 누수를 막기 위해 교차검증 방식으로 만든 기반 예측을 사용합니다.

</details>

**Q3. SHAP 값이 양수와 음수라는 것은 각각 무슨 뜻인가요?**

<details>
<summary>답</summary>

양수는 그 변수가 해당 예측을 기준값보다 위로 밀어 올렸다는 뜻이고, 음수는 아래로 끌어내렸다는 뜻입니다. 예측값은 기준값에 모든 변수의 SHAP 값을 더한 것과 같습니다.

</details>

## 한 줄 정리

> 모델은 공정하게 비교해 고르고, 다양할 때 합치며, SHAP으로 그 판단 근거까지 열어 보는 것이 모델링의 마무리다.
