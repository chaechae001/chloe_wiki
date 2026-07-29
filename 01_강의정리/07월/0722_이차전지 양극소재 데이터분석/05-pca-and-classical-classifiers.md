# PCA와 전통 머신러닝 분류 모델

## 한 줄 요약

PCA로 고차원 피처를 요약하고, 로지스틱 회귀·결정 트리·KNN·랜덤 포레스트·XGBoost의 특성을 비교합니다.

## PCA의 역할

주성분 분석(PCA)은 서로 상관된 피처를 분산을 많이 설명하는 새로운 축으로 변환합니다.

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

X_scaled = StandardScaler().fit_transform(X_train)
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)
print(pca.explained_variance_ratio_)
```

PCA는 시각화와 차원 축소에 유용하지만 주성분의 도메인 해석은 어려워질 수 있습니다.

## 모델별 특징

| 모델 | 장점 | 주의점 |
|---|---|---|
| 로지스틱 회귀 | 빠르고 기준 모델로 좋음 | 비선형 경계 표현이 제한적 |
| 결정 트리 | 해석이 쉽고 비선형 관계 처리 | 깊어지면 과적합 |
| KNN | 단순하고 지역 패턴 반영 | 스케일과 `k`에 민감 |
| 랜덤 포레스트 | 안정적이고 비선형 관계 처리 | 모델 크기와 해석 비용 증가 |
| XGBoost | 강력한 부스팅 성능 | 튜닝과 과적합 관리 필요 |

## 동일 조건에서 비교하기

```python
models = {
    "logistic": LogisticRegression(max_iter=2000),
    "tree": DecisionTreeClassifier(random_state=42),
    "knn": KNeighborsClassifier(n_neighbors=15),
    "forest": RandomForestClassifier(random_state=42),
}

for name, model in models.items():
    model.fit(X_train, y_train)
    pred = model.predict(X_test)
    print(name, accuracy_score(y_test, pred))
```

같은 데이터 분할과 같은 평가지표를 사용해야 공정하게 비교할 수 있습니다.

## KNN의 `k` 선택

- `k`가 너무 작으면 노이즈에 민감해 과적합될 수 있습니다.
- `k`가 너무 크면 지역 구조를 잃어 과소적합될 수 있습니다.
- 검증 또는 교차검증으로 선택합니다.

## 다중 분류 평가

정확도 외에 macro F1과 혼동행렬을 확인합니다. 다중 클래스 ROC-AUC는 `predict_proba()` 결과와 `multi_class="ovr"` 또는 `"ovo"` 설정이 필요합니다.

## 복습 질문 및 답변

**Q1. PCA 전에 표준화가 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	분산이 큰 단위의 피처가 주성분을 독점하는 것을 막고 피처를 같은 기준에서 비교하기 위해서입니다.
</details>

**Q2. 결정 트리와 랜덤 포레스트의 가장 큰 차이는 무엇인가요?**

<details>
<summary>답</summary>
	랜덤 포레스트는 여러 트리를 서로 다른 데이터와 피처로 학습해 평균 또는 투표함으로써 단일 트리의 높은 분산을 줄입니다.
</details>

**Q3. 모델 비교에서 데이터 분할을 고정해야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	분할 난이도의 차이가 아니라 모델 자체의 차이를 비교하고 결과를 재현하기 위해서입니다.
</details>

## 다음 학습

[MLP 분류 모델](06-mlp-classification-and-optimization.md)로 이어집니다.
