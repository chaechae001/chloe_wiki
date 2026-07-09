# 데이터 누수와 파이프라인 — 평가를 믿을 수 있게 만들기

> 데이터 누수는 "시험 전에 답안지를 몰래 본 것"과 같습니다. 점수는 잘 나오지만, 실전에서는 처참하게 무너집니다.

`데이터누수` · `DataLeakage` · `Pipeline` · `전처리순서` · `홀드아웃`

## 핵심요약

- 데이터 누수(Data Leakage)는 예측 시점에 알 수 없는 정보가 학습에 스며들어, 평가 점수가 실제보다 부풀려지는 현상이다.
- 대표 원인은 (1) 분리(split) 전에 전처리·샘플링을 전체 데이터로 수행, (2) 테스트 데이터의 통계가 학습에 유입되는 것이다.
- 해결의 핵심은 순서다. **원본 데이터를 가장 먼저 train/test로 분리**하고, 모든 fit은 train에만 적용한다.
- Scikit-learn의 `Pipeline`은 전처리와 모델을 하나로 묶어, 각 교차검증 fold에서 학습 fold에만 fit되도록 강제해 누수를 구조적으로 차단한다.
- 누수는 "모델이 흔들린다"기보다 "평가 지표 자체를 믿을 수 없게 만든다"는 방식으로 위험하다.

## 1. 데이터 누수란 무엇인가

### 1) 정의

데이터 누수는 머신러닝 모델이 평가용 데이터(test/holdout)의 정보를 학습 과정에서 미리 알게 되는 현상입니다. 이 경우 평가 점수는 비정상적으로 높게 나오지만, 한 번도 보지 못한 실제 운영 데이터에서는 성능이 급락합니다.

### 2) 왜 필요한가 (왜 반드시 막아야 하는가)

- 누수가 있으면 "이 모델은 정확도 90%"라는 보고 자체가 거짓이 됩니다. 진짜 성능을 모르는 채로 배포하는 셈입니다.
- 실무에서는 배포 후 성능이 무너지고 나서야 문제를 알게 되어, 원인 추적 비용이 큽니다.

### 3) 핵심 흐름 재구성 — 잘못된 순서 vs 올바른 순서

**안티 패턴 (누수 발생)**

- 오류 1: 데이터를 train/test로 나누기 **전에** 오버샘플링을 수행. 동일하게 복제된 행이 train과 test 양쪽에 들어가, 모델이 정답을 미리 외웁니다.
- 오류 2: 분리 **전에** 결측치 대체(Imputer)·스케일링(Scaler)을 전체 데이터로 fit. 테스트 데이터의 평균·중앙값 같은 통계가 학습에 유입됩니다.

**베스트 프랙티스 (누수 차단)**

- 개선 1: 어떤 전처리·샘플링보다 **먼저** 원본을 train/test로 분리.
- 개선 2: 오버샘플링은 오직 train에만 적용.
- 개선 3: `Pipeline`을 사용해 모든 `fit`이 train에만 강제 적용되도록 구조화.

### 4) 쉬운 예시

시험공부를 "cm 단위"로 했는데 실제 시험은 "inch"로 나오면 아무리 열심히 해도 점수가 안 나옵니다(전처리 불일치). 반대로 시험 전에 답안지를 몰래 봤다면 모의고사 점수는 만점이지만 실전은 무너집니다(누수). 둘 다 "공부 방식과 시험 방식이 어긋난" 문제입니다.

### 5) 코드 예시 — 누수가 점수를 부풀리는 증거

같은 데이터에서 (A) 분리 전에 오버샘플링한 경우와 (B) 분리 후 train에만 오버샘플링한 경우를, 별도로 떼어 둔 진짜 홀드아웃으로 비교했습니다(공개용 seaborn `titanic` 사용).

```python
import numpy as np, seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
from imblearn.over_sampling import RandomOverSampler

t = sns.load_dataset('titanic')
num = ['pclass', 'age', 'sibsp', 'parch', 'fare']
X = t[num].fillna(t[num].median()); y = t['survived']

# 진짜 홀드아웃을 가장 먼저 떼어 둔다
X_dev, X_hold, y_dev, y_hold = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

def wrong(seed):   # 오류: split 전에 오버샘플링 → 복제 행이 양쪽에 유입
    Xo, yo = RandomOverSampler(random_state=seed).fit_resample(X_dev, y_dev)
    Xtr, Xte, ytr, yte = train_test_split(Xo, yo, test_size=0.25, random_state=seed)
    m = RandomForestClassifier(random_state=seed).fit(Xtr, ytr)
    return accuracy_score(yte, m.predict(Xte)), accuracy_score(y_hold, m.predict(X_hold))

def right(seed):   # 정석: split 먼저 → 오버샘플링은 train에만
    Xtr, Xte, ytr, yte = train_test_split(X_dev, y_dev, test_size=0.25, random_state=seed)
    Xtr2, ytr2 = RandomOverSampler(random_state=seed).fit_resample(Xtr, ytr)
    m = RandomForestClassifier(random_state=seed).fit(Xtr2, ytr2)
    return accuracy_score(yte, m.predict(Xte)), accuracy_score(y_hold, m.predict(X_hold))

for name, fn in [("Wrong", wrong), ("Right", right)]:
    rs = [fn(s) for s in range(20)]
    reported = np.mean([r for r, _ in rs])
    real = np.mean([h for _, h in rs])
    print(f"{name:6s}: 보고된 test acc={reported:.4f} | 실제 홀드아웃 acc={real:.4f} | 과대평가 gap={reported-real:+.4f}")
```

실행 결과(20회 반복 평균):

```
Wrong : 보고된 test acc=0.7809 | 실제 홀드아웃 acc=0.5997 | 과대평가 gap=+0.1812
Right : 보고된 test acc=0.7132 | 실제 홀드아웃 acc=0.5986 | 과대평가 gap=+0.1146
```

### 6) 결과 해석

- **Wrong**: 보고된 정확도는 0.78로 그럴듯하지만, 진짜 홀드아웃 성능은 0.60에 불과합니다. 무려 +0.18의 과대평가입니다. 분리 전 오버샘플링으로 생긴 복제 행이 train과 test에 동시에 들어가, 모델이 test의 정답을 사실상 외웠기 때문입니다.
- **Right**: 과대평가 gap이 +0.11로 더 작습니다. 두 방식의 실제 홀드아웃 성능(0.60)은 비슷한데, 차이는 오직 "보고된 점수를 얼마나 믿을 수 있는가"에 있습니다.
- 핵심 통찰: 누수는 모델의 실제 실력을 크게 바꾸기보다, **평가 지표를 부풀려 신뢰할 수 없게 만든다.** 그래서 배포 전에는 좋아 보이다가 배포 후 무너지는 사고로 이어집니다.

### 7) 한 줄 정리

> 데이터 누수는 평가 점수를 부풀리는 함정이며, 분리를 가장 먼저 하고 fit을 train에만 적용해 막는다.

## 2. Pipeline으로 누수를 구조적으로 막기

### 1) 정의

`Pipeline`은 전처리 단계들과 모델을 하나의 객체로 묶는 도구입니다. `pipeline.fit(X_train)`을 호출하면 내부의 모든 전처리가 train 데이터로만 학습되고, `predict` 시에는 그 학습된 변환이 자동 적용됩니다.

### 2) 왜 필요한가

- 전처리를 수동으로 하면 "test에도 같은 변환 적용하기"를 깜빡하거나, 실수로 전체 데이터에 fit하기 쉽습니다.
- 교차검증에서 특히 중요합니다. Pipeline을 쓰면 각 fold에서 전처리가 학습 fold에만 fit되어, validation fold 정보가 전처리에 새어들지 않습니다.

### 3) 코드 예시

```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipe = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler()),
    ('model', LogisticRegression(max_iter=1000)),
])

pipe.fit(X_train, y_train)     # 모든 fit이 train에만 적용됨
pipe.predict(X_test)           # 학습된 변환을 그대로 재사용
```

### 4) 실무 연결

Scikit-learn 공식 문서도 "자주 하는 실수와 권장 실무(Common Pitfalls)"에서 전처리 불일치와 데이터 누수를 대표 함정으로 꼽고, 그 해법으로 Pipeline을 권합니다. 교차검증·하이퍼파라미터 튜닝과 결합할 때 Pipeline은 사실상 필수입니다(→ [분류 종합 파이프라인](07-classification-pipeline.md)).

## 직접 해보기

1. 스케일러를 전체 데이터로 fit한 뒤 train/test를 나눴다. 어떤 정보가 누수되는가?
2. 오버샘플링을 분리 전에 하면 왜 test 성능이 부풀려지는가?
3. Pipeline이 교차검증에서 특히 유용한 이유를 한 문장으로 설명하라.

<details>
<summary>정답 보기</summary>

1. 스케일러가 전체 데이터(테스트 포함)의 평균·표준편차를 학습하므로, 테스트 데이터의 분포 정보가 변환 기준에 섞여 학습 과정에 유입된다.
2. 오버샘플링이 소수 데이터를 복제하는데, 분리 전에 하면 동일한 복제 행이 train과 test 양쪽에 나뉘어 들어간다. 모델이 train에서 본 행과 똑같은 행을 test에서 맞히므로 점수가 부풀려진다.
3. 각 fold에서 전처리를 학습 fold에만 fit하도록 강제해, validation fold의 정보가 전처리로 새어드는 누수를 자동으로 막아 주기 때문이다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 전처리 불일치 vs 데이터 누수 | train과 test에 다른 변환 적용(성능 하락) vs test 정보가 학습에 유입(성능 과대평가) |
| 분리 전 fit vs 분리 후 fit | 전체 통계가 새어들어 누수 vs train 통계로만 학습해 안전 |
| test set vs 진짜 홀드아웃 | 튜닝·검증 중 반복 참조될 수 있음 vs 마지막 한 번만 보는 최종 점검용 |
| 수동 전처리 vs Pipeline | 실수로 누수 발생 가능 vs fit이 train에만 강제되어 구조적 차단 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [불균형 데이터 처리](03-imbalanced-data.md)
- 다음에 이어지는 개념: [분류 종합 파이프라인](07-classification-pipeline.md)
- 함께 보면 좋은 키워드: `train_test_split`, `fit/transform`, `교차검증`

## 셀프 체크

- [ ] 데이터 누수의 정의를 설명할 수 있다.
- [ ] 누수의 두 가지 대표 원인을 안다.
- [ ] 올바른 전처리 순서(분리 먼저)를 안다.
- [ ] Pipeline이 누수를 막는 원리를 설명할 수 있다.
- [ ] 누수가 평가 지표를 어떻게 왜곡하는지 안다.

### 복습 질문 및 답변

**Q1. 데이터 누수가 있으면 왜 배포 후에야 문제가 드러나는가?**

<details>
<summary>답</summary>

누수는 개발 단계의 평가 점수를 부풀리므로, 배포 전에는 모든 지표가 좋아 보인다. 그러나 실제 운영 데이터에는 누수된 정보가 없으므로 성능이 급락하고, 그제야 문제가 드러난다.

</details>

**Q2. 결측치 대체(Imputer)는 언제 fit해야 하는가?**

<details>
<summary>답</summary>

train/test 분리 이후, train 데이터로만 fit해야 한다. 중앙값·최빈값 같은 대체 기준이 train 통계에서만 계산되어야 test 정보가 새어들지 않는다. Pipeline에 넣으면 이 순서가 자동으로 지켜진다.

</details>

**Q3. "누수는 모델을 흔드는 게 아니라 평가를 흔든다"는 말의 의미는?**

<details>
<summary>답</summary>

누수가 있어도 모델의 실제 예측 실력(진짜 홀드아웃 성능)은 크게 달라지지 않는 경우가 많다. 문제는 보고되는 점수가 실제보다 부풀려져, 그 지표를 신뢰할 수 없게 된다는 데 있다. 즉 위험의 본질은 "성능 저하"가 아니라 "평가의 신뢰성 붕괴"다.

</details>

## 한 줄 정리

> 데이터 누수는 평가 점수를 부풀려 신뢰를 무너뜨리며, 분리를 가장 먼저 하고 Pipeline으로 fit을 train에만 강제해 근본적으로 차단한다.