# 용어집

이번 회차에 등장한 핵심 용어를 쉬운 말로 정리했습니다. 주제별로 나눠 두었으니, 학습 중 막히는 용어를 빠르게 찾아보세요.

## 회귀 기초와 평가

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 회귀(Regression) | 연속적인 숫자 값(가격, 금액 등)을 예측하는 문제. 결과가 범주면 분류 | [1편](01-regression-basics-and-loss.md) | 분류, 손실 함수 |
| 손실 함수(Loss) | 예측이 정답에서 얼마나 벗어났는지를 하나의 숫자로 요약하는 함수. 학습은 이 값을 최소화하는 과정 | [1편](01-regression-basics-and-loss.md) | MSE, 최적화 |
| MSE | 평균제곱오차. 오차를 제곱해 평균 낸 값으로, 큰 오차에 더 큰 벌점을 줌 | [1편](01-regression-basics-and-loss.md) | RMSE, 손실 함수 |
| RMSE | MSE에 제곱근을 씌워 원래 단위로 되돌린 값. 순위는 MSE와 동일 | [2편](02-overfitting-and-metrics.md) | MSE, MAE |
| MAE | 오차의 절대값을 평균 낸 값. 이상치에 상대적으로 둔감함 | [2편](02-overfitting-and-metrics.md) | RMSE, 이상치 |
| 회귀계수(β) | 해당 변수가 1 늘어날 때 예측값이 평균 얼마나 변하는지를 나타내는 값 | [1편](01-regression-basics-and-loss.md) | 절편, 선형 회귀 |
| R²(결정계수) | 모델이 타깃의 변동을 얼마나 설명하는지 나타내는 지표(1에 가까울수록 좋음) | [1편](01-regression-basics-and-loss.md) | 회귀 평가 |
| 과적합(Overfitting) | 훈련 데이터에만 지나치게 맞춰져 새 데이터에서 성능이 떨어지는 현상 | [2편](02-overfitting-and-metrics.md) | 과소적합, 규제 |
| 과소적합(Underfitting) | 모델이 너무 단순해 훈련 데이터의 패턴조차 못 잡는 상태 | [2편](02-overfitting-and-metrics.md) | 과적합, 다항 회귀 |
| 다항 회귀 | 직선 대신 곡선(고차항)으로 데이터를 맞추는 회귀. 차수가 높으면 과적합 위험 | [2편](02-overfitting-and-metrics.md) | 과적합, 차수 |
| 이상치(Outlier) | 다른 값들과 크게 동떨어진 극단적인 관측값. RMSE에 큰 영향을 줌 | [2편](02-overfitting-and-metrics.md) | RMSE, MAE |

## 파이프라인과 검증

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 데이터 누수(Data Leakage) | 테스트 데이터의 정보가 학습에 새어 들어가 성능이 실제보다 좋게 나오는 함정 | [3편](03-data-leakage-and-pipeline.md) | Pipeline, 일반화 |
| Pipeline | 전처리와 모델을 하나로 묶어, 전처리를 train에만 fit하도록 강제하는 도구 | [3편](03-data-leakage-and-pipeline.md) | 데이터 누수, fit/transform |
| fit / transform | 전처리 규칙을 train으로 배우는 것이 fit, test에 적용만 하는 것이 transform | [3편](03-data-leakage-and-pipeline.md) | Pipeline |
| 교차검증(Cross Validation) | 데이터를 여러 조각으로 나눠 성능을 반복 측정해 안정적으로 추정하는 방법 | [3편](03-data-leakage-and-pipeline.md) | KFold, 일반화 |
| KFold | 데이터를 무작위로 K개로 나누는 교차검증 방식 | [3편](03-data-leakage-and-pipeline.md) | StratifiedKFold |
| StratifiedKFold | 타깃 분포를 각 fold에 균등하게 유지하며 나누는 교차검증 방식 | [3편](03-data-leakage-and-pipeline.md) | KFold, qcut |
| paired t-test | 짝지어진 두 조건의 평균 차이가 통계적으로 유의한지 검정하는 방법 | [3편](03-data-leakage-and-pipeline.md) | p-value, 가설검정 |

## 규제와 튜닝

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 규제(Regularization) | 손실에 계수 크기 벌점을 더해 과적합을 억제하는 기법 | [4편](04-regularization-and-hyperparameter-tuning.md) | Ridge, Lasso |
| Ridge(L2) | 계수의 제곱합에 벌점을 주는 규제. 계수를 0 근처로 골고루 줄임 | [4편](04-regularization-and-hyperparameter-tuning.md) | Lasso, alpha |
| Lasso(L1) | 계수의 절대값합에 벌점을 주는 규제. 덜 중요한 계수를 0으로 만들어 변수 선택 효과 | [4편](04-regularization-and-hyperparameter-tuning.md) | Ridge, 희소성 |
| ElasticNet | L1과 L2를 섞은 규제. l1_ratio로 둘의 비율을 조절 | [4편](04-regularization-and-hyperparameter-tuning.md) | Ridge, Lasso |
| 하이퍼파라미터 | 학습으로 배우는 값이 아니라 사람이 학습 전에 정하는 설정값(alpha, 트리 깊이 등) | [4편](04-regularization-and-hyperparameter-tuning.md) | 튜닝, 계수 |
| GridSearch | 정해 둔 후보를 격자처럼 전부 시도해 최적 설정을 찾는 탐색 | [4편](04-regularization-and-hyperparameter-tuning.md) | RandomSearch |
| RandomSearch | 정해진 범위에서 무작위로 후보를 뽑아 시도하는 탐색. 파라미터가 많을 때 효율적 | [4편](04-regularization-and-hyperparameter-tuning.md) | GridSearch |

## 앙상블과 해석

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 리더보드 | 여러 모델을 동일 조건에서 비교해 성능 순으로 정리한 순위표 | [5편](05-ensemble-stacking-and-shap.md) | 교차검증, 모델 비교 |
| 앙상블(Ensemble) | 여러 모델의 예측을 합쳐 더 안정적인 결과를 얻는 기법 | [5편](05-ensemble-stacking-and-shap.md) | Stacking, 다양성 |
| Stacking | 여러 모델의 예측을 다시 메타 모델에 입력해 합치는 앙상블 | [5편](05-ensemble-stacking-and-shap.md) | 앙상블, 메타 모델 |
| 다양성(Diversity) | 기반 모델들이 서로 다르게 예측·오답을 낼수록 앙상블 이득이 커진다는 원리 | [5편](05-ensemble-stacking-and-shap.md) | Stacking |
| SHAP | 각 변수가 개별 예측을 얼마나 밀어 올리거나 내렸는지 수치로 분해하는 설명 기법 | [5편](05-ensemble-stacking-and-shap.md) | 설명 가능성 |
| summary plot | 여러 샘플의 변수 중요도와 작용 방향을 보여주는 SHAP 글로벌 시각화 | [5편](05-ensemble-stacking-and-shap.md) | SHAP, waterfall plot |
| waterfall plot | 예측 한 건이 변수별 기여로 어떻게 조립되는지 보여주는 SHAP 로컬 시각화 | [5편](05-ensemble-stacking-and-shap.md) | SHAP, summary plot |

## 시계열

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 시계열(Time Series) | 시간 순서대로 기록된 데이터로, 과거가 현재에 영향을 줌 | [6편](06-time-series-stationarity-and-acf-pacf.md) | 정상성, ARIMA |
| 정상성(Stationarity) | 평균·분산 등 통계 성질이 시간이 지나도 일정한 상태. 시계열 모델의 전제 | [6편](06-time-series-stationarity-and-acf-pacf.md) | ADF, 차분 |
| ADF 검정 | 정상성을 판정하는 검정. p < 0.05면 정상(귀무가설이 "비정상") | [6편](06-time-series-stationarity-and-acf-pacf.md) | 정상성, 단위근 |
| ACF(자기상관) | 시계열이 자기 과거(여러 시차)와 얼마나 상관되는지 나타내는 함수. MA 차수 판별 | [6편](06-time-series-stationarity-and-acf-pacf.md) | PACF, MA |
| PACF(편자기상관) | 중간 시차의 영향을 뺀 순수한 직접 상관. AR 차수 판별 | [6편](06-time-series-stationarity-and-acf-pacf.md) | ACF, AR |
| AIC | 설명력은 높이되 파라미터가 많으면 벌점을 주는 모델 선택 지표(낮을수록 좋음) | [6편](06-time-series-stationarity-and-acf-pacf.md) | 모델 선택 |
| Box-Jenkins | ACF·PACF로 AR/MA와 차수를 판별하는 시계열 모델링 방법론 | [6편](06-time-series-stationarity-and-acf-pacf.md) | ACF, PACF |
| ARIMA | ARMA에 차분(d)을 더해 비정상 데이터도 다루는 모델. 차수 (p, d, q) | [7편](07-arima-and-residual-diagnostics.md) | ARMA, 차분 |
| 차분(Differencing) | 이웃한 값의 차이를 취해 추세를 제거하고 정상화하는 연산 | [7편](07-arima-and-residual-diagnostics.md) | ARIMA, 정상성 |
| 잔차(Residual) | 실제값에서 예측값을 뺀 나머지. 모델이 설명하지 못한 오차 | [7편](07-arima-and-residual-diagnostics.md) | 백색잡음, 잔차 진단 |
| 백색잡음(White Noise) | 아무 패턴 없는 순수 무작위 오차. 좋은 모델의 잔차가 가져야 할 상태 | [7편](07-arima-and-residual-diagnostics.md) | 잔차, Ljung-Box |
| Ljung-Box 검정 | 잔차에 자기상관이 남았는지 판정하는 검정. p > 0.05면 백색잡음(합격) | [7편](07-arima-and-residual-diagnostics.md) | 백색잡음, 잔차 진단 |
