# 선형 회귀와 로지스틱 회귀 모델

## 한 줄 요약

선형 회귀의 원리와 평가 지표를 이해한 뒤, 범주 예측에는 로지스틱 회귀를 사용해야 하는 이유를 구분합니다.

## 선형 회귀

선형 회귀는 입력의 가중합으로 연속값을 예측합니다.

```python
from sklearn.linear_model import LinearRegression

regressor = LinearRegression()
regressor.fit(X_train, y_train)
y_pred = regressor.predict(X_test)
```

다변수 선형 회귀는 여러 피처를 사용하고, 다중 출력 회귀는 한 번에 여러 연속 목표를 예측합니다.

## 회귀 평가 지표

| 지표 | 해석 |
|---|---|
| MAE | 오차 절댓값의 평균, 직관적 |
| MSE | 큰 오차에 더 큰 벌점 |
| RMSE | 목표값과 같은 단위의 평균 오차 |
| R² | 평균 예측 대비 설명력 |

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

mae = mean_absolute_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred) ** 0.5
r2 = r2_score(y_test, y_pred)
```

## 분류에 선형 회귀를 쓰면 생기는 문제

결정계 라벨을 0, 1, 2로 바꿔 선형 회귀를 적용하면 클래스 사이에 인위적인 거리와 순서를 가정합니다. 예측값도 0.4, 1.7처럼 클래스가 아닌 연속값이 됩니다.

## 다중 로지스틱 회귀

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(
    multi_class="multinomial",
    solver="lbfgs",
    max_iter=2000,
)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
y_prob = model.predict_proba(X_test)
```

다항 로지스틱 회귀는 각 클래스의 확률을 계산하고 가장 큰 확률의 클래스를 선택합니다.

## 피처 선택의 영향

- 중요한 피처가 누락되면 편향이 커지고 성능이 낮아집니다.
- 무관한 피처가 많으면 분산과 계산량이 증가할 수 있습니다.
- 규제 강도 `C`는 과적합과 과소적합의 균형을 조절합니다.

## 복습 질문 및 답변

**Q1. 결정계 분류에 선형 회귀가 적절하지 않은 이유는 무엇인가요?**

<details>
<summary>답</summary>
	범주 라벨을 연속값과 순서가 있는 값처럼 취급하고 클래스 확률을 직접 모델링하지 않기 때문입니다.
</details>

**Q2. RMSE가 MAE보다 큰 오차에 민감한 이유는 무엇인가요?**

<details>
<summary>답</summary>
	오차를 제곱한 뒤 평균을 내므로 큰 오차가 결과에 훨씬 크게 반영되기 때문입니다.
</details>

**Q3. `predict_proba()`는 무엇을 반환하나요?**

<details>
<summary>답</summary>
	각 샘플이 클래스별로 속할 확률을 반환하며, 다중 분류 평가와 의사결정 임계값 분석에 사용할 수 있습니다.
</details>

## 다음 학습

[PCA와 전통 분류 모델](05-pca-and-classical-classifiers.md)에서 여러 모델을 비교합니다.
