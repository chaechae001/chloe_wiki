# 데이터 누수와 파이프라인·교차검증 — 정직한 성능 추정의 기술

> 전처리를 잘못하면 모델이 시험 정답을 미리 훔쳐본 셈이 되어, 실전에서 무너집니다. 이 글은 데이터 누수를 원천 차단하는 파이프라인과, 성능을 안정적으로 추정하는 교차검증을 다룹니다.

`데이터누수` · `Pipeline` · `교차검증` · `StratifiedKFold` · `paired t-test`

## 핵심요약

- 데이터 누수(Data Leakage)는 테스트 데이터의 정보가 학습 과정에 새어 들어가, 성능이 실제보다 좋게 나오는 함정이다.
- 전처리(스케일링·특징선택 등)는 반드시 **train에만 fit**하고 test에는 transform만 적용해야 한다.
- `Pipeline`은 전처리와 모델을 하나로 묶어 이 규칙을 자동으로 지켜 준다.
- 교차검증은 데이터를 여러 번 나눠 성능을 반복 측정해, 한 번의 우연에 휘둘리지 않게 한다.
- KFold는 무작위 분할, StratifiedKFold는 타깃 분포를 각 fold에 균등하게 유지하는 분할이다.

## 1. 데이터 누수와 파이프라인

### 1) 정의

데이터 누수는 모델이 학습할 때 **원래는 몰라야 할 테스트 데이터의 정보**를 미리 사용해 버리는 것입니다. 시험 전에 답안지를 훔쳐본 학생이 그 시험에서만 높은 점수를 받는 것과 같습니다. 배포 후 진짜 새 데이터에서는 성능이 뚝 떨어집니다.

### 2) 왜 필요한가

가장 흔한 누수는 **전체 데이터로 스케일링이나 특징선택을 한 뒤에 train/test를 나누는** 실수입니다. 이러면 스케일러가 계산한 평균·표준편차나, 특징선택이 고른 변수에 이미 테스트 데이터의 정보가 녹아 있습니다. 그 결과 테스트 성능이 부풀려지고, 우리는 그 부풀린 숫자를 믿고 잘못된 모델을 배포하게 됩니다. 누수를 막아야 성능 추정이 정직해집니다.

### 3) 핵심 흐름 재구성

올바른 순서는 항상 이렇습니다.

```text
1. 먼저 train / test를 분리한다.
2. 전처리(scaler, 특징선택 등)를 train에만 fit 한다.
3. 같은 전처리를 test에는 transform 만 적용한다(fit 금지).
4. 모델을 train으로 학습하고 test로 평가한다.
```

이 순서를 매번 손으로 지키는 건 실수하기 쉽습니다. 그래서 사이킷런의 `Pipeline`을 씁니다. 파이프라인은 전처리와 모델을 하나의 객체로 묶고, `fit`을 부르면 내부적으로 train에만 전처리를 fit하고, `predict`를 부르면 test에 transform만 적용합니다. 규칙을 코드 구조가 강제해 주는 셈입니다.

### 4) 쉬운 예시

요리 대회에 비유하면, 심사용 재료(test)를 미리 맛보고 간을 맞춘 요리사는 그 심사에서만 유리합니다. 공정하려면 연습 재료(train)로만 간을 정하고, 심사 재료에는 그 정해진 간을 그대로 적용해야 합니다. 파이프라인은 "연습 재료로만 간을 정한다"를 자동으로 지켜 주는 규칙집입니다.

### 5) 코드 예시

누수가 극적으로 드러나는 상황은 **특징선택**입니다. 정답(y)을 이용해 전체 데이터에서 유용한 변수를 고르면, 그 선택 자체에 테스트 정보가 새어 듭니다. 노이즈 변수 200개를 섞은 데이터로 확인해 봅니다.

```python
import numpy as np, seaborn as sns
from sklearn.feature_selection import SelectKBest, f_regression
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
np.random.seed(0)

df = sns.load_dataset('tips'); y = df['tip'].values
X_real = df[['total_bill', 'size']].values
X_noise = np.random.randn(len(df), 200)       # 의미 없는 노이즈 변수 200개
X = np.hstack([X_real, X_noise])

# [누수] 전체 데이터에서 정답 y로 특징선택 → 그다음 분리
X_sel = SelectKBest(f_regression, k=10).fit(X, y).transform(X)
Xtr, Xte, ytr, yte = train_test_split(X_sel, y, test_size=0.3, random_state=42)
rmse_leak = np.sqrt(mean_squared_error(yte, Ridge(alpha=1.0).fit(Xtr, ytr).predict(Xte)))

# [올바름] 파이프라인 안에서 train에만 fit
pipe = Pipeline([('sel', SelectKBest(f_regression, k=10)),
                 ('model', Ridge(alpha=1.0))])
Xtr2, Xte2, ytr2, yte2 = train_test_split(X, y, test_size=0.3, random_state=42)
pipe.fit(Xtr2, ytr2)
rmse_clean = np.sqrt(mean_squared_error(yte2, pipe.predict(Xte2)))

print(f'[누수]   Test RMSE = {rmse_leak:.4f}')
print(f'[올바름] Test RMSE = {rmse_clean:.4f}')
```

```text
[누수]   Test RMSE = 0.8724
[올바름] Test RMSE = 0.9667
```

### 코드 목적

같은 데이터·같은 모델인데, 전처리 순서만 잘못해도 성능이 얼마나 낙관적으로 왜곡되는지 보여줍니다.

### 코드 흐름

1. 실제 신호 2개에 노이즈 변수 200개를 섞는다.
2. 누수 버전: 전체 데이터에서 정답을 보고 변수 10개를 고른 뒤 분리한다.
3. 올바른 버전: 파이프라인으로 묶어 train에서만 변수를 고르게 한다.
4. 두 버전의 test RMSE를 비교한다.

### 실행 결과 해석

누수 버전의 test RMSE는 0.8724로, 올바른 버전 0.9667보다 **약 0.09 낮게(=더 좋아 보이게)** 나옵니다. 순전히 노이즈였던 변수들이 "전체 데이터 기준으로는" 우연히 정답과 잘 맞아 선택되었고, 그 선택에 테스트 정보가 새어 든 결과입니다. 이 0.8724를 믿고 배포하면 실제 환경에서는 0.97 수준(혹은 그 이하)으로 성능이 떨어집니다. 파이프라인은 이 착시를 원천 차단합니다.

### 실무 연결

실무 데이터 파이프라인에는 결측치 대치, 스케일링, 인코딩, 특징선택 등 학습이 필요한 단계가 여럿 있습니다. 이 모든 단계를 파이프라인으로 묶으면, 교차검증이나 배포 시에도 누수 없이 일관되게 동작합니다. 캐글 상위권 노트북이 거의 예외 없이 파이프라인을 쓰는 이유이기도 합니다.

### 6) 헷갈리는 점

"스케일링은 데이터 전체에 하면 편하지 않나?"가 대표적 오해입니다. 편하지만 그게 바로 누수입니다. 스케일러의 평균·표준편차는 오직 train으로만 계산해야 합니다.

### 7) 한 줄 정리

> 전처리는 반드시 train에만 fit하며, 파이프라인이 이 규칙을 코드 구조로 강제해 데이터 누수를 막는다.

## 2. 교차검증 — KFold vs StratifiedKFold

### 1) 정의

교차검증(Cross Validation)은 데이터를 여러 조각(fold)으로 나눠, 번갈아 가며 하나를 검증용으로 두고 나머지로 학습하는 과정을 반복해 성능을 여러 번 측정하는 방법입니다. KFold는 데이터를 무작위로 K개로 나누고, StratifiedKFold는 **타깃의 분포를 각 fold에 비슷하게 유지**하며 나눕니다.

### 2) 왜 필요한가

train/test를 한 번만 나누면, 그 한 번의 분할이 우연히 쉽거나 어렵게 나올 수 있습니다. 교차검증은 여러 번 나눠 평균과 표준편차를 보므로, 성능을 더 안정적으로 추정하고 "이 모델이 얼마나 일관적인가"까지 알 수 있습니다.

### 3) 핵심 흐름 재구성

분류에서는 타깃이 불균형할 때 StratifiedKFold가 거의 필수입니다. 어떤 fold에 특정 클래스가 하나도 안 들어가는 사고를 막기 때문입니다. 회귀에서는 타깃이 연속값이라 층화가 바로 안 되지만, 타깃을 구간(bin)으로 나눈 뒤 그 구간을 기준으로 층화하면 각 fold의 타깃 분포를 고르게 맞출 수 있습니다. 두 방식 중 어느 것이 실제로 더 나은지는 데이터마다 다르므로, **통계 검정으로 비교**하는 것이 정석입니다.

### 4) 쉬운 예시

반 학생 30명을 5개 조로 나눠 조별 평가를 한다고 합시다. 무작위로 나누면(KFold) 어떤 조에 우연히 상위권만, 다른 조에 하위권만 몰릴 수 있습니다. 성적대를 고르게 섞어 나누면(StratifiedKFold) 모든 조가 비슷한 실력 분포를 갖습니다. 후자가 평가를 더 공정하게 만듭니다.

### 5) 코드 예시

`tips`의 팁 금액을 5구간으로 나눠 층화 기준으로 삼고, KFold와 StratifiedKFold의 fold별 RMSE를 비교합니다.

```python
import numpy as np, pandas as pd, seaborn as sns
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
from sklearn.model_selection import KFold, StratifiedKFold
from sklearn.metrics import mean_squared_error

df = sns.load_dataset('tips').copy()
df['bill_per_person'] = df['total_bill'] / df['size']
X = df[['total_bill', 'size', 'bill_per_person']].values
y = df['tip'].values
df['tip_bin'] = pd.qcut(df['tip'], q=5, labels=False, duplicates='drop')  # 연속 타깃 구간화
y_bin = df['tip_bin'].values

def fold_rmses(splitter, y_strat=None):
    rmses = []
    splits = splitter.split(X, y_strat) if y_strat is not None else splitter.split(X)
    for tri, tei in splits:
        p = Pipeline([('s', StandardScaler()), ('m', Ridge(alpha=1.0))]).fit(X[tri], y[tri])
        rmses.append(np.sqrt(mean_squared_error(y[tei], p.predict(X[tei]))))
    return np.array(rmses)

r_kf  = fold_rmses(KFold(5, shuffle=True, random_state=42))
r_skf = fold_rmses(StratifiedKFold(5, shuffle=True, random_state=42), y_bin)
print(f'KFold        평균={r_kf.mean():.4f}  std={r_kf.std():.4f}')
print(f'StratifiedKF 평균={r_skf.mean():.4f}  std={r_skf.std():.4f}')
```

```text
KFold        평균=1.0263  std=0.1368
StratifiedKF 평균=1.0066  std=0.1403
```

### 코드 목적

두 교차검증 방식이 이 데이터에서 실제로 성능 차이를 내는지 확인합니다.

### 코드 흐름

1. 연속 타깃(tip)을 `qcut`으로 5개 구간으로 나눠 층화 기준을 만든다.
2. 파이프라인(스케일러+Ridge)을 fold마다 새로 학습·평가하는 함수를 정의한다.
3. KFold와 StratifiedKFold로 각각 fold별 RMSE를 구한다.
4. 평균과 표준편차를 비교한다.

### 실행 결과 해석

이 데이터에서는 StratifiedKFold의 평균 RMSE(1.0066)가 KFold(1.0263)보다 약간 낮습니다. 하지만 표준편차를 보면 두 방식의 fold별 변동이 비슷한 수준이라, 이 차이가 **통계적으로 의미 있는 차이인지 우연인지**는 단발 결과로 단정할 수 없습니다. 그래서 여러 시드로 반복해 통계 검정을 해야 합니다.

### 6) 헷갈리는 점

"StratifiedKFold가 언제나 낫다"는 오해가 있습니다. 타깃이 심하게 불균형하거나 소수 구간이 있을 때는 층화가 확실히 유리하지만, 데이터가 충분하고 분포가 고르면 두 방식의 차이가 거의 없을 수 있습니다. 항상 검증으로 확인해야 합니다.

### 7) 한 줄 정리

> 교차검증은 성능을 여러 번 재서 안정적으로 추정하고, 타깃 분포가 중요하면 층화(Stratified)로 각 fold를 고르게 맞춘다.

## 코드로 보기 — paired t-test로 두 검증 방식 비교

두 방식의 차이가 우연인지 판단하려면, 같은 시드 조건에서 짝지어 비교하는 **대응표본 t-검정(paired t-test)**을 씁니다.

```python
from scipy import stats
# 30개 시드에서 각 방식의 평균 RMSE를 짝지어 수집한 뒤 검정
pk = [fold_rmses(KFold(5, shuffle=True, random_state=s)).mean() for s in range(30)]
ps = [fold_rmses(StratifiedKFold(5, shuffle=True, random_state=s), y_bin).mean() for s in range(30)]
t_stat, p_val = stats.ttest_rel(pk, ps)
print(f'KFold 평균={np.mean(pk):.4f}, StratifiedKFold 평균={np.mean(ps):.4f}')
print(f't={t_stat:.4f}, p-value={p_val:.4e}')
```

```text
KFold 평균=1.0280, StratifiedKFold 평균=1.0296
t=-0.4970, p-value=6.2291e-01
```

### 코드 목적

단발 결과의 우연을 배제하고, 두 검증 방식 사이에 진짜 성능 차이가 있는지를 통계적으로 판정합니다.

### 실행 결과 해석

30개 시드로 평균을 내니 두 방식의 성능이 거의 같아졌고(1.0280 vs 1.0296), p-value가 0.62로 0.05보다 훨씬 큽니다. 따라서 **"두 방식 사이에 통계적으로 유의미한 성능 차이가 없다"**는 결론입니다. 앞의 단발 실험에서 보인 작은 차이는 우연이었던 것입니다. 이것이 단발 결과에 흥분하지 않고 반복 검정으로 확인해야 하는 이유입니다.

### 실무 연결

"방식 A가 B보다 좋아 보인다"는 주장을 데이터로 검증할 때 paired t-test는 표준 도구입니다. A/B 테스트, 모델 버전 비교, 전처리 방식 비교 등에서 "차이가 진짜인가 우연인가"를 가려 줍니다.

## 직접 해보기

1. 전체 데이터로 스케일링한 뒤 train/test를 나누면 왜 문제가 되나요?
2. 회귀 문제에서 연속형 타깃을 StratifiedKFold로 층화하려면 먼저 무엇을 해야 하나요?
3. 두 검증 방식의 paired t-test 결과 p-value가 0.62일 때 내릴 수 있는 결론은?

<details>
<summary>정답 보기</summary>

1. 스케일러가 test 데이터의 평균·표준편차까지 반영해 계산되므로, 테스트 정보가 학습에 새어 듭니다(데이터 누수). 성능이 실제보다 좋게 나와 배포 후 성능 하락을 부릅니다.
2. 연속형 타깃을 `pd.qcut` 등으로 구간(bin)으로 나눠 범주형 기준을 만든 뒤, 그 구간을 층화 기준으로 사용합니다.
3. p-value(0.62)가 0.05보다 크므로 귀무가설(두 방식에 차이 없음)을 기각하지 못합니다. 즉 두 방식 사이에 통계적으로 유의미한 성능 차이가 없다고 판단합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| fit vs transform | 전처리 규칙을 train으로 배우는 게 fit, test에 적용만 하는 게 transform |
| 데이터 누수 vs 정상 전처리 | 분리 전 전처리는 누수, 분리 후 train에만 fit하면 정상 |
| KFold vs StratifiedKFold | 무작위 분할 vs 타깃 분포를 각 fold에 균등 유지 |
| 단발 검증 vs 교차검증 | 한 번 나눠 보는 것 vs 여러 번 나눠 평균·편차까지 보는 것 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [과적합과 회귀 평가지표](02-overfitting-and-metrics.md)
- 다음에 이어지는 개념: [규제와 하이퍼파라미터 튜닝](04-regularization-and-hyperparameter-tuning.md)
- 함께 보면 좋은 키워드: `cross_val_score`, `qcut`, `가설검정`

## 셀프 체크

- [ ] 데이터 누수가 무엇이고 왜 위험한지 설명할 수 있다.
- [ ] 전처리를 train에만 fit해야 하는 이유를 안다.
- [ ] 파이프라인이 누수를 막는 원리를 이해한다.
- [ ] KFold와 StratifiedKFold의 차이를 안다.
- [ ] paired t-test 결과를 해석할 수 있다.

### 복습 질문 및 답변

**Q1. 파이프라인은 데이터 누수를 어떻게 막아 주나요?**

<details>
<summary>답</summary>

파이프라인은 전처리와 모델을 하나로 묶어, `fit`을 부르면 전처리를 train에만 fit하고 `predict`를 부르면 test에는 transform만 적용합니다. 개발자가 순서를 실수할 여지 없이 코드 구조가 규칙을 강제합니다.

</details>

**Q2. 교차검증을 쓰면 단발 train/test 분할보다 무엇이 좋아지나요?**

<details>
<summary>답</summary>

데이터를 여러 번 나눠 성능을 반복 측정하므로, 한 번의 분할이 우연히 쉽거나 어렵게 나오는 편향을 줄입니다. 평균으로 성능을, 표준편차로 모델의 일관성을 함께 파악할 수 있습니다.

</details>

**Q3. StratifiedKFold가 항상 KFold보다 좋은가요?**

<details>
<summary>답</summary>

아닙니다. 타깃이 불균형하거나 소수 구간이 있을 때는 층화가 유리하지만, 데이터가 충분하고 분포가 고르면 차이가 거의 없을 수 있습니다. 실제 예시에서도 30개 시드 검정 결과 유의미한 차이가 없었습니다. 데이터마다 검증으로 확인해야 합니다.

</details>

## 한 줄 정리

> 정직한 성능 추정의 두 기둥은 누수를 막는 파이프라인과, 우연을 걸러 내는 교차검증·통계 검정이다.
