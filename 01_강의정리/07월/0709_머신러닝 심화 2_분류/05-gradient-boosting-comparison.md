# 그래디언트 부스팅 3형제 — XGBoost·LightGBM·CatBoost 비교

> 트리를 순서대로 쌓아 실수를 보완하는 GBDT 계열. 캐글 상위권의 단골 무기.

`부스팅` · `GBDT` · `XGBoost` · `LightGBM` · `CatBoost`

## 핵심요약

- 세 모델 모두 트리를 순차적으로 쌓아 앞선 트리의 오차를 보완하는 GBDT 계열이다.
- 공통 뼈대는 같지만, 트리를 키우는 방식과 범주형·속도 처리에서 차이가 난다.
- LightGBM은 리프 중심 성장으로 대개 빠르고, CatBoost는 범주형 처리에 강하다.
- 조기 종료(early stopping)로 검증 성능이 더 안 오르면 학습을 멈춰 과적합과 시간을 아낀다.
- 어느 하나가 항상 최고인 것은 아니며, 데이터에 따라 순위가 바뀐다.

---

## 1. 부스팅과 GBDT의 뼈대

### 1) 정의

부스팅(Boosting)은 약한 학습기(주로 얕은 트리)를 **순차적으로** 쌓되, 각 단계에서 앞 모델이 틀린 부분을 집중적으로 보완하도록 학습하는 앙상블 기법이다. GBDT(Gradient Boosting Decision Tree)는 이 보완을 손실함수의 그래디언트(기울기)를 따라 수행한다.

### 2) 왜 필요한가

단일 트리는 약하고 불안정하다. 부스팅은 트리들을 협력시켜 정형(tabular) 데이터에서 매우 높은 성능을 낸다. 실무의 표 형태 데이터에서 부스팅 계열이 상위권을 차지하는 경우가 많다.

### 3) 핵심 흐름 재구성

- 첫 트리가 대략 예측한다.
- 남은 오차(잔차)를 다음 트리가 예측하도록 학습한다.
- 이 과정을 반복하며 예측을 조금씩 더한다.
- `learning_rate`가 각 트리의 반영 비율을 조절한다(작을수록 신중, 트리 더 필요).

배깅(랜덤 포레스트)이 트리들을 **병렬·독립**으로 만들어 평균 내는 것과 달리, 부스팅은 **순차·의존**적으로 오차를 이어받아 보완한다.

### 4) 쉬운 예시

시험 오답노트를 떠올리자. 1차에서 틀린 문제를 골라 집중 보완하고, 2차에서 또 틀린 부분을 다시 보완한다. 매 라운드가 이전 라운드의 약점을 메우는 방식이 부스팅이다.

### 5) 코드 예시

```python
from xgboost import XGBClassifier

xgb = XGBClassifier(
    n_estimators=500,      # 최대 트리 수
    learning_rate=0.1,     # 각 트리 반영 비율
    max_depth=3,           # 개별 트리 깊이(약한 학습기)
    eval_metric='logloss',
    early_stopping_rounds=30,
    random_state=42,
)
xgb.fit(X_tr, y_tr, eval_set=[(X_val, y_val)], verbose=False)
print("best_iteration:", xgb.best_iteration)
```

`early_stopping_rounds=30`은 검증 성능이 30번 연속 개선되지 않으면 학습을 멈춘다는 뜻이다.

### 6) 헷갈리는 점

- 부스팅은 순차 학습이라 트리 순서가 의미를 가진다. 배깅처럼 아무 순서로 평균 내지 않는다.
- `n_estimators`는 "최대"일 뿐, 조기 종료가 실제 트리 수를 줄일 수 있다.

### 7) 한 줄 정리

> 부스팅은 앞 트리의 오차를 뒤 트리가 이어받아 보완하는 순차 앙상블이다.

---

## 2. 세 모델의 차이

### 1) 정의

세 모델은 GBDT라는 같은 뼈대 위에서 트리 성장 방식·범주형 처리·속도 최적화가 다르다.

| 항목 | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| 트리 성장 | 레벨 중심(균형) | 리프 중심(불균형) | 대칭(oblivious) 트리 |
| 강점 | 안정적·표준 | 대용량·빠른 속도 | 범주형 자동 처리 |
| 범주형 | 직접 인코딩 필요 | 지원(설정) | 강력한 내장 처리 |

### 2) 왜 필요한가

데이터 크기, 범주형 변수 비중, 속도 요구가 다르면 유리한 모델도 달라진다. 세 모델의 성격을 알면 상황에 맞게 고르거나 여러 개를 함께 시도할 수 있다.

### 3) 핵심 흐름 재구성

- **LightGBM**: 손실을 가장 많이 줄이는 리프를 먼저 키우는 리프 중심 성장으로 대체로 빠르지만, 얕은 데이터에선 과적합에 주의해야 한다.
- **CatBoost**: 범주형 변수를 별도 인코딩 없이 잘 다루고, 대칭 트리로 안정적이다.
- **XGBoost**: 오래 검증된 표준으로, 규제 옵션이 풍부하다.

### 4) 쉬운 예시

같은 요리(부스팅)를 만드는 세 요리사라고 보자. 한 명은 빠른 손놀림(LightGBM), 한 명은 재료 손질(범주형)에 능숙(CatBoost), 한 명은 기본기가 탄탄한 베테랑(XGBoost)이다. 손님(데이터)에 따라 누가 더 잘 맞는지 달라진다.

### 5) 코드 예시

```python
import lightgbm as lgb
from lightgbm import LGBMClassifier
from catboost import CatBoostClassifier

lgbm = LGBMClassifier(n_estimators=500, learning_rate=0.1, max_depth=3,
                      random_state=42, verbose=-1)
lgbm.fit(X_tr, y_tr, eval_set=[(X_val, y_val)],
         callbacks=[lgb.early_stopping(30, verbose=False)])

cat = CatBoostClassifier(iterations=500, learning_rate=0.1, depth=3,
                         random_state=42, verbose=False, early_stopping_rounds=30)
cat.fit(X_tr, y_tr, eval_set=(X_val, y_val))
```

### 6) 헷갈리는 점

- 라이브러리마다 조기 종료 지정 방식이 다르다(XGBoost는 생성자 인자, LightGBM은 콜백, CatBoost는 인자).
- 같은 `depth`라도 트리 성장 방식이 달라 결과가 다르게 나온다.

### 7) 한 줄 정리

> 세 모델은 GBDT 뼈대는 같지만 트리 성장·범주형·속도 최적화에서 갈린다.

---

## 코드로 보기 — 같은 데이터로 3형제 나란히 비교

```python
import time
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, roc_auc_score
from xgboost import XGBClassifier
from lightgbm import LGBMClassifier
import lightgbm as lgb
from catboost import CatBoostClassifier

df = sns.load_dataset('titanic')[
    ['survived','age','fare','pclass','sex','sibsp','parch']].dropna().copy()
df['sex'] = (df['sex'] == 'female').astype(int)
X = df[['age','fare','pclass','sex','sibsp','parch']]
y = df['survived']

# train/val/test = 60/20/20
X_trv, X_te, y_trv, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
X_tr, X_val, y_tr, y_val = train_test_split(X_trv, y_trv, test_size=0.25,
                                            stratify=y_trv, random_state=42)

def report(name, model, best_iter):
    acc = accuracy_score(y_te, model.predict(X_te))
    auc = roc_auc_score(y_te, model.predict_proba(X_te)[:, 1])
    print(f"{name:9s} best_iter={best_iter:<4} acc={acc:.3f} auc={auc:.3f}")

xgb = XGBClassifier(n_estimators=500, learning_rate=0.1, max_depth=3,
                    eval_metric='logloss', early_stopping_rounds=30, random_state=42)
xgb.fit(X_tr, y_tr, eval_set=[(X_val, y_val)], verbose=False)
report("XGBoost", xgb, xgb.best_iteration)

lgbm = LGBMClassifier(n_estimators=500, learning_rate=0.1, max_depth=3,
                      random_state=42, verbose=-1)
lgbm.fit(X_tr, y_tr, eval_set=[(X_val, y_val)],
         callbacks=[lgb.early_stopping(30, verbose=False)])
report("LightGBM", lgbm, lgbm.best_iteration_)

cat = CatBoostClassifier(iterations=500, learning_rate=0.1, depth=3,
                         random_state=42, verbose=False, early_stopping_rounds=30)
cat.fit(X_tr, y_tr, eval_set=(X_val, y_val))
report("CatBoost", cat, cat.get_best_iteration())
```

### 코드 목적

같은 데이터·같은 하이퍼파라미터에서 세 부스팅 모델을 조기 종료와 함께 비교해, 성능과 학습 트리 수의 차이를 확인하기 위한 코드다.

### 코드 흐름

1. 데이터를 train/val/test 세 갈래로 나눈다.
2. 각 모델을 val로 모니터링하며 조기 종료로 학습한다.
3. test에서 정확도와 AUC를 측정한다.
4. best_iteration으로 몇 개의 트리에서 멈췄는지 본다.

### 실행 결과 해석

```text
XGBoost  best_iter=154  acc=0.790 auc=0.837
LightGBM best_iter=79   acc=0.811 auc=0.846
CatBoost best_iter=104  acc=0.797 auc=0.855
```

이 데이터에서는 정확도는 LightGBM, AUC는 CatBoost가 가장 높았고, 세 모델이 500개까지 다 쓰지 않고 조기 종료로 멈췄다. 순위는 데이터·하이퍼파라미터에 따라 얼마든지 바뀔 수 있으므로, "이 조건에서의 결과"로 읽어야 한다.

### 실무 연결

수요 예측, 이탈 예측, 신용 스코어링 등 표 형태 데이터의 예측에서 세 모델을 함께 시도해 가장 잘 맞는 것을 고르는 방식이 일반적이다. 조기 종료는 학습 시간과 과적합을 동시에 줄여준다.

---

## 직접 해보기

1. 부스팅과 배깅(랜덤 포레스트)의 가장 큰 구조적 차이는?
2. `early_stopping_rounds=30`은 정확히 무슨 조건에서 학습을 멈추게 하는가?
3. `learning_rate`를 낮추면 필요한 트리 수는 대체로 어떻게 변하는가?

<details>
<summary>정답 보기</summary>

1. 배깅은 트리들을 독립·병렬로 만들어 평균 낸다. 부스팅은 트리들을 순차·의존적으로 쌓아 앞 트리의 오차를 뒤 트리가 보완한다.
2. 검증 세트 성능이 30번 연속으로 개선되지 않으면 학습을 중단한다. 그 시점 이전의 최적 트리 수를 best_iteration으로 기억한다.
3. 각 트리의 반영 비율이 작아지므로 같은 성능에 도달하려면 더 많은 트리가 필요해진다. 대신 과적합 위험은 줄고 보통 더 안정적이다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 부스팅 vs 배깅 | 순차 보완이 부스팅, 병렬 평균이 배깅 |
| 레벨 성장 vs 리프 성장 | XGBoost는 균형 성장, LightGBM은 손실 큰 리프 우선 |
| n_estimators vs best_iteration | 전자는 최대 허용 트리 수, 후자는 조기 종료로 실제 쓰인 수 |
| learning_rate 큼 vs 작음 | 크면 빠르지만 과적합 위험, 작으면 신중하지만 트리 더 필요 |

## 연결되는 개념

- 이전에 알면 좋은 개념: 의사결정나무 — 분할·불순도·과적합
- 다음에 이어지는 개념: 하이퍼파라미터 튜닝과 교차검증
- 함께 보면 좋은 키워드: `앙상블`, `조기종료`, `잔차`

## 셀프 체크

- [ ] GBDT의 순차 보완 구조를 설명할 수 있다.
- [ ] 세 모델의 성격 차이를 안다.
- [ ] 조기 종료의 작동 조건을 설명할 수 있다.
- [ ] train/val/test 각각의 역할을 안다.
- [ ] learning_rate와 트리 수의 관계를 안다.

### 복습 질문 및 답변

**Q1. 검증(Validation) 세트는 왜 필요한가?**

<details>
<summary>답</summary>

조기 종료 시점을 정하고 하이퍼파라미터를 고르기 위해서다. 이 판단에 test를 쓰면 test가 오염되어 최종 성능을 정직하게 측정할 수 없으므로, train과 test 사이에 별도 검증 세트를 둔다.

</details>

**Q2. LightGBM이 대체로 빠른 이유는?**

<details>
<summary>답</summary>

손실을 가장 많이 줄이는 리프를 우선 키우는 리프 중심 성장과, 히스토그램 기반 분할 등 속도 최적화 덕분이다. 다만 데이터가 작으면 과적합에 주의해야 한다.

</details>

**Q3. "CatBoost가 항상 최고"라고 말하면 안 되는 이유는?**

<details>
<summary>답</summary>

모델 순위는 데이터 특성, 범주형 비중, 하이퍼파라미터에 따라 달라진다. 어떤 데이터에서 좋았다고 다른 데이터에서도 최고인 것은 아니므로, 여러 모델을 비교해 선택해야 한다.

</details>

## 한 줄 정리

> XGBoost·LightGBM·CatBoost는 같은 부스팅 뼈대의 세 변형으로, 데이터에 맞게 비교해 고르고 조기 종료로 과적합을 관리한다.
