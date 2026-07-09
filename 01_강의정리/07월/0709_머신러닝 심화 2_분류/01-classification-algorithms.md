# 분류 알고리즘 한눈에 보기 — 로지스틱 회귀·SVM·나이브 베이즈·KNN

> "이 손님은 이탈할까, 남을까?" 같은 예/아니오 문제를 푸는 네 가지 대표 도구를 비교합니다.

`분류` · `로지스틱회귀` · `시그모이드` · `SVM` · `KNN`

## 핵심요약

- 분류는 정답이 숫자(회귀)가 아니라 **범주(클래스)** 인 문제를 푸는 지도학습이다.
- 로지스틱 회귀는 직선 결과를 시그모이드로 눌러 0~1 확률로 바꾸고, 보통 0.5를 기준으로 나눈다.
- SVM은 두 클래스 사이 여백(margin)이 가장 넓어지는 경계를 찾는다.
- 나이브 베이즈는 "특성들이 서로 독립"이라는 단순 가정을 두고 확률을 곱해 계산한다.
- KNN은 학습이랄 게 없고, 예측할 때 가까운 이웃 K명에게 다수결로 물어본다.

---

## 1. 분류란 무엇인가

### 1) 정의

분류(Classification)는 입력 데이터를 미리 정해진 몇 개의 **범주 중 하나로 배정**하는 문제다. 생존/사망, 스팸/정상, 개/고양이/새처럼 답이 "얼마"가 아니라 "무엇"인 경우다.

### 2) 왜 필요한가

현실의 의사결정 상당수가 예/아니오 형태다. 이 고객에게 쿠폰을 줄지, 이 거래가 사기인지, 이 사진이 불량품인지 판단하는 일이 전부 분류다. 회귀가 "얼마나"를 예측한다면 분류는 "어느 쪽"을 예측한다.

### 3) 핵심 흐름 재구성

분류 알고리즘은 결국 **특성 공간에 결정 경계(decision boundary)를 긋는 방법**의 차이다. 같은 데이터라도 직선으로 긋는지, 곡선으로 긋는지, 이웃을 기준으로 나누는지가 알고리즘마다 다르다. 이번 글에서는 네 가지 접근을 같은 데이터(공개 titanic)로 비교한다.

- 로지스틱 회귀: 선형 경계 + 확률 변환
- SVM: 여백 최대화 경계(커널로 비선형도 가능)
- 나이브 베이즈: 확률 기반, 조건부 독립 가정
- KNN: 거리 기반, 게으른 학습(lazy learning)

### 4) 쉬운 예시

우산을 챙길지 결정한다고 하자. 로지스틱 회귀는 "습도·구름양을 점수로 합산해 비 올 확률 70%"라고 말한다. KNN은 "지난 5번 중 비슷한 날씨 4번이 비였으니 비"라고 다수결로 말한다. 같은 결론이라도 도달하는 방식이 다르다.

### 5) 코드 예시

```python
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# 공개 데이터(titanic)에서 필요한 열만 사용
df = sns.load_dataset('titanic')[['survived','age','fare','pclass','sex']].dropna().copy()
df['sex'] = (df['sex'] == 'female').astype(int)   # 문자 → 0/1

X = df[['age','fare','pclass','sex']].values
y = df['survived'].values

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)

# 스케일 통일: age(0~80)와 fare(0~500)의 크기 차이를 맞춘다
sc = StandardScaler()
X_tr_s = sc.fit_transform(X_tr)   # train에서만 기준 학습
X_te_s = sc.transform(X_te)       # test는 그 기준을 그대로 적용

clf = LogisticRegression(max_iter=1000, random_state=42)
clf.fit(X_tr_s, y_tr)
print("Accuracy:", round(accuracy_score(y_te, clf.predict(X_te_s)), 3))
```

실행 결과:

```text
Accuracy: 0.767
```

### 6) 헷갈리는 점

- `fit`은 **train 데이터로만** 한다. 스케일러도 마찬가지다. test까지 넣고 `fit`하면 정보 누수가 된다.
- 스케일링이 필요한 알고리즘(SVM·KNN·로지스틱 회귀)과 상대적으로 덜 민감한 알고리즘(트리 계열)이 있다.

### 7) 한 줄 정리

> 분류는 결정 경계를 긋는 문제이고, 알고리즘마다 경계를 긋는 철학이 다르다.

---

## 2. 로지스틱 회귀와 시그모이드

### 1) 정의

로지스틱 회귀는 이름은 회귀지만 **분류** 알고리즘이다. 선형 결합(β₀ + β₁x₁ + …)을 계산한 뒤 그 값을 시그모이드 함수에 통과시켜 0~1 사이 확률로 바꾼다.

### 2) 왜 필요한가

선형 회귀 결과는 음수도, 1보다 큰 값도 나올 수 있어 "확률"로 쓰기 어렵다. 시그모이드는 어떤 실수든 0~1 사이로 눌러주므로 확률 해석이 가능해진다.

### 3) 핵심 흐름 재구성

시그모이드 함수는 다음과 같이 정의된다.

```text
σ(z) = 1 / (1 + e^(−z))
```

- z가 0이면 σ = 0.5
- z가 커질수록 1에 가까워지고, 작아질수록 0에 가까워진다
- 보통 σ ≥ 0.5(즉 z ≥ 0)이면 1, 아니면 0으로 분류한다

### 4) 쉬운 예시

시험 점수를 합격 확률로 바꾼다고 생각하자. 아주 낮은 점수는 확률 0에, 아주 높은 점수는 확률 1에 가깝게, 애매한 중간은 0.5 근처로 부드럽게 이어지는 S자 곡선이 시그모이드다.

### 5) 코드 예시

```python
from scipy.special import expit   # expit이 곧 시그모이드

for z in [-2, -1, 0, 1, 2]:
    print(f"z={z:+d} -> sigmoid={expit(z):.3f}")
```

실행 결과:

```text
z=-2 -> sigmoid=0.119
z=-1 -> sigmoid=0.269
z=+0 -> sigmoid=0.500
z=+1 -> sigmoid=0.731
z=+2 -> sigmoid=0.881
```

z가 0에서 멀어질수록 확률이 0 또는 1로 빠르게 수렴하는 것을 볼 수 있다.

### 6) 헷갈리는 점

- 0.5는 고정값이 아니라 **선택 가능한 임계값**이다. 상황에 따라 0.3이나 0.7로 바꿀 수 있다(다음 글에서 다룬다).
- "회귀"라는 이름 때문에 연속값을 예측한다고 오해하기 쉽지만, 최종 출력은 클래스다.

### 7) 한 줄 정리

> 로지스틱 회귀는 선형 점수를 시그모이드로 확률화한 뒤 임계값으로 잘라 분류한다.

---

## 3. SVM · 나이브 베이즈 · KNN

### 1) 정의

- **SVM(서포트 벡터 머신)**: 두 클래스를 가르는 경계 중, 양쪽 가장 가까운 점(서포트 벡터)까지의 여백이 최대가 되는 경계를 찾는다. 커널을 쓰면 곡선 경계도 만든다.
- **나이브 베이즈**: 베이즈 정리를 기반으로, 각 특성이 서로 독립이라는 "순진한(naive)" 가정 아래 확률을 곱해 분류한다.
- **KNN(K-최근접 이웃)**: 새 점 주변에서 가장 가까운 K개 이웃의 다수 클래스로 예측한다.

### 2) 왜 필요한가

데이터 모양에 따라 잘 맞는 알고리즘이 다르다. 선형으로 나뉘면 로지스틱·선형 SVM이, 복잡하게 얽히면 RBF 커널 SVM이나 KNN이 유리할 수 있다. 여러 후보를 알아야 문제에 맞게 고를 수 있다.

### 3) 핵심 흐름 재구성

베이즈 정리는 다음과 같다.

```text
P(A|B) = P(B|A) × P(A) / P(B)
```

나이브 베이즈는 "관측된 특성(B)이 주어졌을 때 클래스(A)일 확률"을 이 식으로 뒤집어 계산한다. 특성이 여러 개면 각각 독립이라 가정하고 확률을 곱한다. 가정은 단순하지만 텍스트 분류 등에서 의외로 잘 작동한다.

KNN에서 K는 이웃 수다. K가 작으면 노이즈에 민감하고, 크면 경계가 부드러워지지만 둔감해진다.

### 4) 쉬운 예시

새로 이사 온 동네에서 어느 정당을 지지할지 추측한다고 하자. KNN이라면 "가장 가까운 이웃 5집 중 3집이 A당 → A당"이라 말한다. K=1이면 옆집 한 곳에만 휘둘리고, K=31이면 동네 전체 분위기에 가까워진다.

### 5) 코드 예시

```python
from sklearn.svm import SVC
from sklearn.naive_bayes import GaussianNB
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

# 앞서 만든 X_tr_s, X_te_s, y_tr, y_te 재사용
for name, clf in [
    ("SVM(rbf)",   SVC(kernel='rbf', C=1.0, gamma='scale', probability=True, random_state=42)),
    ("NaiveBayes", GaussianNB()),
]:
    clf.fit(X_tr_s, y_tr)
    print(f"{name:12s} Acc={accuracy_score(y_te, clf.predict(X_te_s)):.3f}")

for k in [3, 5, 15, 31]:
    knn = KNeighborsClassifier(n_neighbors=k)
    knn.fit(X_tr_s, y_tr)
    print(f"KNN k={k:2d}    Acc={accuracy_score(y_te, knn.predict(X_te_s)):.3f}")
```

실행 결과:

```text
SVM(rbf)     Acc=0.791
NaiveBayes   Acc=0.744
KNN k= 3    Acc=0.781
KNN k= 5    Acc=0.758
KNN k=15    Acc=0.786
KNN k=31    Acc=0.763
```

### 6) 헷갈리는 점

- KNN은 학습 단계가 사실상 없고, 예측할 때 계산량이 몰린다. 데이터가 크면 느리다.
- SVM의 `C`는 클수록 오차를 덜 허용(과적합 위험↑), `gamma`는 클수록 경계가 복잡해진다.

### 7) 한 줄 정리

> SVM은 여백, 나이브 베이즈는 확률 곱, KNN은 이웃 다수결 — 서로 다른 세 가지 결정 방식이다.

---

## 코드로 보기 — 네 알고리즘 정확도 한 번에 비교

```python
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.naive_bayes import GaussianNB
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

df = sns.load_dataset('titanic')[['survived','age','fare','pclass','sex']].dropna().copy()
df['sex'] = (df['sex'] == 'female').astype(int)
X = df[['age','fare','pclass','sex']].values
y = df['survived'].values

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)
sc = StandardScaler()
X_tr_s, X_te_s = sc.fit_transform(X_tr), sc.transform(X_te)

models = {
    "LogReg":     LogisticRegression(max_iter=1000, random_state=42),
    "SVM":        SVC(kernel='rbf', probability=True, random_state=42),
    "NaiveBayes": GaussianNB(),
    "KNN(15)":    KNeighborsClassifier(n_neighbors=15),
}
for name, m in models.items():
    m.fit(X_tr_s, y_tr)
    print(f"{name:11s} {accuracy_score(y_te, m.predict(X_te_s)):.3f}")
```

### 코드 목적

같은 전처리·같은 분할에서 네 알고리즘의 기본 성능을 나란히 비교해, 데이터에 어떤 접근이 잘 맞는지 감을 잡기 위한 코드다.

### 코드 흐름

1. 공개 데이터에서 특성과 정답을 준비한다.
2. train/test로 나누고 스케일을 통일한다.
3. 네 모델을 같은 조건으로 학습한다.
4. test 정확도를 출력해 비교한다.

### 실행 결과 해석

이 데이터에서는 RBF 커널 SVM(0.791)이 가장 높았고 나이브 베이즈(0.744)가 상대적으로 낮았다. 다만 이 수치는 특성 선택·스케일·랜덤 분할에 따라 달라지므로, "SVM이 항상 최고"라는 결론이 아니라 "이 조건에서의 한 결과"로 읽어야 한다.

### 실무 연결

이탈 예측, 사기 탐지, 스팸 필터처럼 예/아니오를 판단하는 서비스에서 여러 알고리즘을 빠르게 비교해 베이스라인을 잡는 작업에 그대로 쓰인다.

---

## 직접 해보기

1. 로지스틱 회귀에서 시그모이드 출력이 0.5보다 클 때 예측 클래스는 무엇인가?
2. KNN에서 K를 1에서 31로 키우면 결정 경계는 어떻게 변하는가?
3. 스팸 필터를 만드는데 "정상 메일을 스팸으로 잘못 막는 것"이 특히 치명적이라면, 임계값을 어떻게 조정하는 게 합리적일까?

<details>
<summary>정답 보기</summary>

1. 1(양성)로 분류한다. 시그모이드 ≥ 0.5는 z ≥ 0을 의미하며, 기본 결정 규칙에서 양성으로 본다.
2. K가 커질수록 개별 이웃의 영향이 줄어 경계가 부드러워지고 노이즈에 둔감해진다. 대신 너무 크면 세밀한 패턴을 놓친다.
3. 스팸으로 판정하는 임계값을 높인다(예: 0.5 → 0.8). 확실할 때만 차단하므로 정상 메일 오차단(거짓 양성)이 줄어든다. 대신 일부 스팸은 통과할 수 있다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 회귀 vs 분류 | 연속값 예측은 회귀, 범주 예측은 분류 |
| 로지스틱 회귀 vs 선형 회귀 | 시그모이드로 확률화해 분류하면 로지스틱, 직선으로 값 예측하면 선형 |
| SVM의 C vs gamma | C는 오차 허용 정도, gamma는 경계 복잡도 |
| KNN 작은 K vs 큰 K | 작으면 민감·과적합, 크면 부드러움·둔감 |

## 연결되는 개념

- 이전에 알면 좋은 개념: 지도학습과 train/test 분리
- 다음에 이어지는 개념: 분류 평가지표 — 정확도·정밀도·재현율·F1
- 함께 보면 좋은 키워드: `결정경계`, `스케일링`, `확률예측`

## 셀프 체크

- [ ] 분류와 회귀의 차이를 설명할 수 있다.
- [ ] 시그모이드가 왜 필요한지 말할 수 있다.
- [ ] SVM·나이브 베이즈·KNN의 결정 방식을 구분할 수 있다.
- [ ] KNN에서 K의 역할을 설명할 수 있다.
- [ ] 스케일링이 필요한 알고리즘을 예로 들 수 있다.

### 복습 질문 및 답변

**Q1. 로지스틱 회귀가 "회귀"라는 이름에도 분류로 쓰이는 이유는?**

<details>
<summary>답</summary>

선형 결합 결과를 시그모이드로 0~1 확률로 바꾼 뒤 임계값으로 클래스를 나누기 때문이다. 내부에 선형 회귀 구조가 있어 이름은 회귀지만, 최종 출력은 범주다.

</details>

**Q2. KNN이 "게으른 학습(lazy learning)"이라 불리는 이유는?**

<details>
<summary>답</summary>

학습 단계에서 모델 파라미터를 따로 만들지 않고 데이터를 저장만 해두었다가, 예측 시점에 이웃을 찾아 계산하기 때문이다. 학습은 빠르지만 예측이 느리다.

</details>

**Q3. 나이브 베이즈의 "순진한(naive)" 가정이란 무엇이고, 왜 그래도 잘 작동할까?**

<details>
<summary>답</summary>

모든 특성이 서로 독립이라고 가정하는 것이다. 현실에서는 특성끼리 상관이 있어 이 가정이 완벽히 맞지 않지만, 확률의 상대적 크기 비교만 필요할 때가 많아 텍스트 분류 등에서 실용적으로 잘 맞는다.

</details>

## 한 줄 정리

> 분류는 결정 경계를 긋는 문제이며, 로지스틱 회귀·SVM·나이브 베이즈·KNN은 그 경계를 각각 확률·여백·확률곱·이웃 다수결로 그리는 서로 다른 도구다.
