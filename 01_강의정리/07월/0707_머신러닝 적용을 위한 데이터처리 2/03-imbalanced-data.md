# 불균형 데이터 처리 — 정확도의 함정과 리샘플링

> 사기 탐지, 불량 검출, 희귀 질병 진단처럼 "찾아야 할 것이 드문" 문제에서는 정확도 95%가 오히려 실패의 신호일 수 있습니다.

`불균형데이터` · `언더샘플링` · `오버샘플링` · `SMOTE` · `재현율`

## 핵심요약

- 클래스 비율이 크게 치우친 데이터에서는 모델이 다수 클래스로만 찍어도 정확도가 높게 나오는 착시가 생긴다.
- 불균형 비율(IR)이 10:1 이상(소수 10% 미만)이면 심각한 불균형으로 보고 리샘플링을 검토한다.
- 언더샘플링은 다수 클래스를 줄이고(정보 손실 위험), 오버샘플링은 소수 클래스를 늘린다(과적합 위험).
- SMOTE는 소수 클래스 사이를 선형 보간해 "새로운 가상 데이터"를 합성하므로 단순 복제보다 일반화에 유리하다.
- 불균형 문제의 진짜 성능 지표는 정확도가 아니라 소수 클래스의 재현율(Recall)과 F1-score다.

## 1. 데이터 불균형과 정확도의 함정

### 1) 정의

데이터 불균형(imbalanced data)은 예측 대상 클래스의 비율이 한쪽으로 심하게 치우친 상태를 말합니다. 예를 들어 정상 거래 95%, 사기 거래 5%처럼 관심 대상(소수 클래스)이 드문 경우입니다.

### 2) 왜 필요한가

불균형을 방치하면 모델의 손실 함수가 다수 클래스의 오분류 비용을 더 크게 반영해, 모델이 항상 다수 클래스로만 예측해도 전체 정확도가 높게 나옵니다. 정작 잡아내야 할 소수 클래스(사기·불량·질병)를 거의 놓치는데도 성적표만 화려해지는 것입니다.

### 3) 핵심 흐름 재구성

불균형의 심각성을 수치로 표현하는 기본 지표가 **불균형 비율(IR, Imbalance Ratio)** 입니다.

```
IR = (다수 클래스 수) / (소수 클래스 수)
```

- 예: IR ≈ 18:1 → 소수 데이터 1개당 다수 데이터가 18개 쌓인다는 뜻.
- 실무 기준: IR ≥ 10:1(소수 10% 미만)이면 심각한 불균형으로 보고 반드시 샘플링 전처리를 검토한다.

### 4) 쉬운 예시

100명 중 3명만 특정 질병 환자인 검진 데이터를 생각해 보세요. "모두 정상"이라고만 답해도 정확도는 97%입니다. 하지만 환자 3명을 전부 놓쳤으니, 정작 필요한 일은 하나도 못 한 모델입니다.

### 5) 코드 예시 — 베이스라인의 착시

인위적으로 소수 클래스 약 5%인 데이터를 만들어 로지스틱 회귀를 학습시킨 결과입니다(공개용 `make_classification` 사용).

```python
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

X, y = make_classification(
    n_samples=2000, n_features=2, n_redundant=0, n_clusters_per_class=1,
    weights=[0.95, 0.05], flip_y=0.01, random_state=42)

print("클래스 분포:", dict(pd.Series(y).value_counts().sort_index()))

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)
model = LogisticRegression(max_iter=1000, random_state=42).fit(X_tr, y_tr)
print(classification_report(y_te, model.predict(X_te), target_names=['C0', 'C1']))
```

실행 결과:

```
클래스 분포: {0: 1893, 1: 107}
              precision    recall  f1-score   support

          C0       0.96      0.99      0.97       568
          C1       0.54      0.22      0.31        32

    accuracy                           0.95       600
   macro avg       0.75      0.60      0.64       600
weighted avg       0.94      0.95      0.94       600
```

### 6) 결과 해석 (매우 중요)

- **정확도 0.95의 함정**: 전체 정확도는 95%로 훌륭해 보이지만, 원래 데이터에 C0가 95% 섞여 있어 "무조건 C0"라고만 해도 도달하는 가짜 성적표입니다.
- **C1 재현율 0.22**: 테스트의 소수 클래스 32개 중 실제로 잡아낸 건 약 7개뿐(32 × 0.22)입니다. 나머지는 정상으로 흘려보냈습니다. 실무라면 대형 사고입니다.
- **C1 F1-score 0.31**: 소수 클래스의 진짜 실력을 대변하는 F1이 바닥이라는 것은, 모델이 소수 클래스의 특징을 거의 학습하지 못했다는 증거입니다.

### 7) 한 줄 정리

> 불균형 데이터에서 정확도는 착시를 만들며, 소수 클래스의 재현율과 F1로 실력을 판단해야 한다.

## 2. 언더샘플링 — 다수를 줄이기

### 1) 정의

언더샘플링(under-sampling)은 다수 클래스의 데이터를 줄여 소수 클래스와 비율을 맞추는 방법입니다.

### 2) 주요 기법

- **Random Under-sampling**: 다수 클래스를 무작위로 삭제. 가장 단순한 기준선. 데이터가 대량으로 사라져 정보 손실이 큼.
- **Tomek Links**: 서로 다른 클래스이면서 서로에게 가장 가까운 이웃인 쌍을 찾아, 그중 다수 클래스 데이터만 제거. 경계선의 노이즈만 정밀 제거해 결정 경계를 선명하게 함(비율을 1:1로 맞추지는 않음).
- **CNN (Condensed Nearest Neighbour)**: 소수 클래스는 모두 남기고, 다수 클래스 중 공간 구조를 대표하는 최소한의 핵심점만 남김. 중복 데이터를 압축.

### 3) 헷갈리는 점

Tomek Links는 "균형을 맞추는" 기법이 아니라 "경계선 노이즈를 지우는" 기법입니다. 그래서 실행 후에도 데이터가 아주 조금만 줄어드는 것이 정상입니다.

### 4) 한 줄 정리

> 언더샘플링은 다수를 줄여 균형을 맞추지만, 버려지는 데이터에 담긴 패턴까지 잃을 수 있다.

## 3. 오버샘플링 — 소수를 늘리기

### 1) 정의

오버샘플링(over-sampling)은 소수 클래스의 데이터를 늘려 다수 클래스와 비율을 맞추는 방법입니다.

### 2) 주요 기법

- **Random Over-sampling**: 기존 소수 데이터를 무작위로 복제. 좌표는 그대로 겹쳐 쌓이므로, 특정 위치의 가중치만 무거워져 과적합 위험이 큼.
- **SMOTE**: 소수 클래스 내부에서 이웃(k-NN)을 찾아 두 점 사이를 선형 보간해 새로운 가상 좌표를 만듦. 겹쳐 쌓는 게 아니라 "면적을 확장"하므로 일반화에 유리.
- **Borderline-SMOTE**: 경계면(위험 지대)에 있는 소수 샘플을 우선 골라 그 주변에서 합성. 결정 경계 근처를 집중 보강.

### 3) SMOTE의 핵심 수식

새 가상 데이터는 소수 샘플 x와 그 이웃 x_nn 사이의 직선 위에 만들어집니다.

```
x_new = x + λ × (x_nn − x)      (단, λ 는 0 과 1 사이 난수)
```

즉 두 소수 점을 잇는 선분 위 어딘가에 새 점을 찍는 것입니다. 그래서 산점도에서 소수 점들 사이가 그물망처럼 메워지는 모습이 관찰됩니다.

### 4) 헷갈리는 점

- **Random Over vs SMOTE**: 전자는 같은 좌표에 데이터를 겹쳐 쌓아 밀도만 진해지고, 후자는 사이 공간에 새 좌표를 만들어 면적을 넓힙니다.
- **SMOTE의 한계**: 소수 점끼리만 이웃을 맺어 보간하므로, 다수 클래스 한가운데 외톨이로 있던 소수 점이 있으면 그 주변에 노이즈성 가상 데이터가 생길 수 있습니다.

### 5) 한 줄 정리

> 오버샘플링은 소수를 늘려 균형을 맞추며, SMOTE는 단순 복제 대신 보간으로 새 데이터를 합성해 과적합을 완화한다.

## 코드로 보기 — 여섯 가지 기법 성능 비교

동일한 불균형 데이터에 각 리샘플링을 적용하고, 소수 클래스(C1)의 재현율·F1·정확도를 비교했습니다.

```python
from imblearn.under_sampling import RandomUnderSampler, TomekLinks
from imblearn.over_sampling import RandomOverSampler, SMOTE
from sklearn.metrics import recall_score, f1_score, accuracy_score

def evaluate(name, X_fit, y_fit):
    m = LogisticRegression(max_iter=1000, random_state=42).fit(X_fit, y_fit)
    pred = m.predict(X_te)
    return dict(method=name, n_train=len(y_fit),
                recall_c1=round(recall_score(y_te, pred), 3),
                f1_c1=round(f1_score(y_te, pred), 3),
                accuracy=round(accuracy_score(y_te, pred), 3))

rows = [evaluate("Baseline", X_tr, y_tr)]
for name, sampler in [("RandomUnder", RandomUnderSampler(random_state=42)),
                      ("TomekLinks", TomekLinks()),
                      ("RandomOver", RandomOverSampler(random_state=42)),
                      ("SMOTE", SMOTE(random_state=42))]:
    Xr, yr = sampler.fit_resample(X_tr, y_tr)
    rows.append(evaluate(name, Xr, yr))

print(pd.DataFrame(rows).to_string(index=False))
```

실행 결과:

```
     method  n_train  recall_c1  f1_c1  accuracy
   Baseline     1400      0.219  0.311     0.948
RandomUnder      150      0.875  0.415     0.868
 TomekLinks     1384      0.219  0.304     0.947
 RandomOver     2650      0.875  0.400     0.860
      SMOTE     2650      0.875  0.421     0.872
```

### 코드 목적

`fit_resample`이라는 공통 API로 여러 기법을 같은 조건에서 돌려, 어떤 방법이 소수 클래스 검출력을 얼마나 끌어올리는지 정량 비교하는 것이 목적입니다.

### 실행 결과 해석

- **Baseline**: 정확도 0.948로 가장 높지만 C1 재현율은 0.219뿐. 전형적인 다수 클래스 편향입니다.
- **RandomUnder**: 학습 데이터가 1400→150으로 급감했지만 재현율은 0.875로 폭등. 대신 정확도가 0.868로 하락(오탐 증가의 대가).
- **TomekLinks**: 경계 노이즈만 조금 지워 재현율 변화가 거의 없음. 단순 선형 분류기에서는 효과가 제한적입니다.
- **SMOTE**: 재현율 0.875를 확보하면서 F1이 0.421로 가장 높음. 정확도 하락도 상대적으로 완만해, 균형 잡힌 선택지입니다.

핵심 교훈: **정확도가 아니라 재현율·F1로 봐야 진짜 개선이 보인다.** RandomUnder와 SMOTE는 재현율이 같지만, 정보 손실이 적은 SMOTE의 F1이 더 높습니다.

### 실무 연결

금융 사기 탐지, 제조 불량 검출, 의료 진단, 이상 거래 모니터링 등 "드문 사건을 놓치면 치명적인" 영역 전반에 적용됩니다. 이런 곳에서는 소수 클래스를 놓친 오류(양성을 음성으로 판정)가 그 반대보다 훨씬 위험하므로 재현율이 최우선 지표가 됩니다.

## 직접 해보기

1. 소수 클래스가 전체의 4%인 데이터에서 "모두 다수 클래스"라고 예측하는 모델의 정확도는? 이 정확도가 무의미한 이유는?
2. Random Over-sampling과 SMOTE는 둘 다 소수를 늘린다. 산점도에서 관찰되는 차이는 무엇인가?
3. 사기 탐지에서 "정상을 사기로 오판"과 "사기를 정상으로 오판" 중 더 치명적인 쪽은? 이때 우선해야 할 지표는?

<details>
<summary>정답 보기</summary>

1. 96%(다수 클래스 비율만큼). 하지만 소수 클래스를 하나도 못 잡아내므로, 정작 필요한 예측을 전혀 못 하는 무의미한 정확도다. 재현율·F1로 평가해야 한다.
2. Random Over는 기존 좌표에 데이터를 그대로 복제해 겹쳐 쌓아 밀도(색)만 진해진다. SMOTE는 소수 점들 사이 선분 위에 새 좌표를 합성해, 소수 클래스가 차지하는 면적 자체가 넓어진다.
3. "사기를 정상으로 오판"(양성을 음성으로 놓침)이 더 치명적이다. 이 오류를 줄이려면 소수 클래스의 재현율(Recall)을 우선해야 한다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 언더샘플링 vs 오버샘플링 | 다수를 줄임(정보 손실) vs 소수를 늘림(과적합 위험) |
| Random Over vs SMOTE | 기존 데이터 복제로 밀도만 증가 vs 보간으로 새 데이터 합성해 면적 확장 |
| 정확도 vs 재현율 | 전체 맞힌 비율(불균형에서 착시) vs 실제 양성 중 잡아낸 비율(불균형의 진짜 지표) |
| Tomek Links vs Random Under | 경계 노이즈만 정밀 제거 vs 다수를 무작위 대량 삭제 |
| 정밀도 vs 재현율 | 사기라 예측한 것 중 진짜 비율 vs 진짜 사기 중 잡아낸 비율 |

## 연결되는 개념

- 이전에 알면 좋은 개념: [API로 데이터 수집하기](02-api-crawling.md)
- 다음에 이어지는 개념: [데이터 누수와 파이프라인](04-data-leakage.md)
- 함께 보면 좋은 키워드: `Recall`, `F1-score`, `fit_resample`, `Confusion Matrix`

## 셀프 체크

- [ ] 불균형에서 정확도가 왜 착시인지 설명할 수 있다.
- [ ] 불균형 비율(IR)을 계산하고 해석할 수 있다.
- [ ] 언더샘플링과 오버샘플링의 장단점을 구분한다.
- [ ] SMOTE가 단순 복제와 어떻게 다른지 안다.
- [ ] 불균형에서 우선해야 할 평가 지표를 안다.

### 복습 질문 및 답변

**Q1. 소수 클래스 5% 데이터에서 정확도 95%가 나왔다. 좋은 모델인가?**

<details>
<summary>답</summary>

아니다. "무조건 다수 클래스"라고만 답해도 95%가 나오는 착시일 수 있다. 소수 클래스의 재현율과 F1을 확인하기 전까지는 판단할 수 없다.

</details>

**Q2. SMOTE의 λ(람다)는 무엇을 의미하는가?**

<details>
<summary>답</summary>

소수 샘플 x와 이웃 x_nn을 잇는 선분 위에서, 새 점을 어느 위치에 찍을지 정하는 0과 1 사이의 난수다. λ가 0에 가까우면 x 근처에, 1에 가까우면 x_nn 근처에 가상 데이터가 생긴다.

</details>

**Q3. 재현율이 같은데 F1이 다른 두 기법 중 무엇을 고르겠는가?**

<details>
<summary>답</summary>

F1이 더 높은 쪽을 고른다. F1은 정밀도와 재현율의 조화평균이라, 재현율이 같다면 F1이 높은 쪽이 오탐(정밀도 하락)이 적어 균형이 낫다는 뜻이다. 위 실험에서는 RandomUnder와 SMOTE의 재현율이 같지만 SMOTE의 F1이 더 높다.

</details>

## 한 줄 정리

> 불균형 데이터는 정확도의 착시를 만들므로 재현율·F1로 평가하고, 언더/오버샘플링과 SMOTE로 소수 클래스 검출력을 끌어올린다.
