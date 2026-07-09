# 용어집

이번 묶음(분류·부스팅·앙상블)에서 등장한 핵심 용어를 비전공자도 이해할 수 있게 쉬운 말로 정리했습니다. 검색이 쉽도록 관련 개념을 함께 묶었습니다.

## 분류 알고리즘

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 분류(Classification) | 데이터를 정해진 범주 중 하나로 배정하는 지도학습. 답이 "얼마"가 아니라 "무엇"이다 | [분류 알고리즘](01-classification-algorithms.md) | 회귀, 결정 경계 |
| 결정 경계(Decision Boundary) | 클래스를 나누는 기준선. 알고리즘마다 직선·곡선·이웃 기반으로 다르게 긋는다 | [분류 알고리즘](01-classification-algorithms.md) | 분류, SVM |
| 로지스틱 회귀 | 선형 점수를 시그모이드로 확률화한 뒤 임계값으로 나누는 분류기. 이름은 회귀지만 분류 | [분류 알고리즘](01-classification-algorithms.md) | 시그모이드, 임계값 |
| 시그모이드(Sigmoid) | 어떤 실수든 0~1 사이로 눌러 확률로 바꾸는 S자 함수. σ(z)=1/(1+e^(−z)) | [분류 알고리즘](01-classification-algorithms.md) | 로지스틱 회귀, 확률 |
| SVM | 두 클래스 사이 여백(margin)이 가장 넓어지는 경계를 찾는 분류기. 커널로 곡선 경계도 가능 | [분류 알고리즘](01-classification-algorithms.md) | 결정 경계, 커널 |
| 나이브 베이즈 | 특성이 서로 독립이라 가정하고 베이즈 정리로 확률을 곱해 분류하는 방법 | [분류 알고리즘](01-classification-algorithms.md) | 베이즈 정리, 조건부 독립 |
| KNN | 예측 시 가까운 이웃 K개의 다수 클래스로 판단하는 거리 기반·게으른 학습 | [분류 알고리즘](01-classification-algorithms.md) | 거리, 게으른 학습 |
| 스케일링(Scaling) | 특성마다 다른 크기·단위를 맞추는 전처리. 거리·확률 기반 모델에 중요 | [분류 알고리즘](01-classification-algorithms.md) | 표준화, 데이터 누수 |

## 평가지표

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 혼동행렬(Confusion Matrix) | 실제와 예측을 교차한 표. TP·TN·FP·FN으로 나뉘며 모든 지표의 출발점 | [평가지표](02-classification-metrics.md) | TP/FP/FN/TN, 정밀도 |
| 정밀도(Precision) | 양성이라 예측한 것 중 진짜 양성 비율. TP/(TP+FP). "예측의 신뢰도" | [평가지표](02-classification-metrics.md) | 재현율, F1 |
| 재현율(Recall) | 실제 양성 중 잡아낸 비율. TP/(TP+FN). "놓치지 않는 능력" | [평가지표](02-classification-metrics.md) | 정밀도, TPR |
| F1 점수 | 정밀도와 재현율의 조화평균. 한쪽이 낮으면 크게 떨어진다 | [평가지표](02-classification-metrics.md) | 정밀도, 재현율 |
| 정확도(Accuracy) | 전체 중 맞힌 비율. 불균형 데이터에서는 착시를 일으킨다 | [평가지표](02-classification-metrics.md) | 불균형 데이터, F1 |
| 불균형 데이터 | 클래스 비율이 크게 치우친 데이터. 정확도만 보면 위험하다 | [평가지표](02-classification-metrics.md) | 정확도의 함정, 재현율 |

## ROC · 임계값

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 임계값(Threshold) | 확률을 0/1로 나누는 기준선. 기본 0.5지만 상황에 맞게 바꾼다 | [ROC·AUC](03-roc-auc-threshold.md) | predict_proba, 트레이드오프 |
| ROC 커브 | 임계값을 훑으며 (FPR, TPR) 궤적을 그린 곡선. 왼쪽 위에 가까울수록 좋다 | [ROC·AUC](03-roc-auc-threshold.md) | AUC, TPR/FPR |
| AUC | ROC 커브 아래 넓이. 임계값과 무관한 판별력 지표. 0.5는 무작위, 1.0은 완벽 | [ROC·AUC](03-roc-auc-threshold.md) | ROC 커브, 판별력 |
| TPR / FPR | 실제 양성 중 잡은 비율(TPR=재현율), 실제 음성 중 오탐 비율(FPR) | [ROC·AUC](03-roc-auc-threshold.md) | ROC 커브, 재현율 |

## 트리 · 부스팅

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 의사결정나무 | 특성 조건으로 데이터를 계속 나눠 규칙을 만드는 해석 쉬운 모델 | [의사결정나무](04-decision-tree.md) | 불순도, 과적합 |
| 불순도(Impurity) | 한 노드에 여러 클래스가 섞인 정도. 순수하면 0 | [의사결정나무](04-decision-tree.md) | 지니, 엔트로피 |
| 지니 지수(Gini) | 불순도 척도. 1−(P(양성)²+P(음성)²). 계산이 가볍다 | [의사결정나무](04-decision-tree.md) | 엔트로피, 정보이득 |
| 엔트로피(Entropy) | 로그 기반 불순도 척도. −Σ pᵢ log₂(pᵢ). 반반이면 1.0 | [의사결정나무](04-decision-tree.md) | 지니, 정보이득 |
| 정보이득(Information Gain) | 분할 전후 불순도가 줄어든 정도. 클수록 좋은 분할 | [의사결정나무](04-decision-tree.md) | 불순도, 분할 |
| 과적합(Overfitting) | 훈련 데이터를 지나치게 외워 새 데이터 성능이 떨어지는 현상 | [의사결정나무](04-decision-tree.md) | max_depth, 일반화 |
| 부스팅(Boosting) | 약한 트리를 순차로 쌓아 앞 트리의 오차를 뒤 트리가 보완하는 앙상블 | [부스팅 3형제](05-gradient-boosting-comparison.md) | GBDT, 배깅 |
| GBDT | Gradient Boosting Decision Tree. 손실의 기울기를 따라 트리를 쌓는 부스팅 | [부스팅 3형제](05-gradient-boosting-comparison.md) | 부스팅, 잔차 |
| 조기 종료(Early Stopping) | 검증 성능이 더 안 오르면 학습을 멈춰 과적합·시간을 줄이는 기법 | [부스팅 3형제](05-gradient-boosting-comparison.md) | 검증 세트, 과적합 |
| learning_rate | 각 트리를 얼마나 반영할지 정하는 값. 작으면 신중하지만 트리 더 필요 | [부스팅 3형제](05-gradient-boosting-comparison.md) | 부스팅, n_estimators |

## 튜닝 · 검증 · 앙상블

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 하이퍼파라미터 | 학습 전에 사람이 정하는 설정값(트리 깊이, 학습률 등) | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | 파라미터, GridSearch |
| 교차검증(Cross Validation) | 데이터를 K조각으로 나눠 돌아가며 검증해 성능을 안정적으로 추정 | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | StratifiedKFold, 튜닝 |
| StratifiedKFold | 클래스 비율을 유지하며 폴드를 나누는 교차검증 방식 | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | 교차검증, 불균형 데이터 |
| GridSearch | 하이퍼파라미터 조합을 모두 교차검증으로 시험해 최적을 찾는 방법 | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | RandomSearch, 교차검증 |
| 파이프라인(Pipeline) | 전처리와 모델을 하나로 묶어 순서를 재현하고 누수를 막는 도구 | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | ColumnTransformer, 데이터 누수 |
| 데이터 누수(Data Leakage) | 알 수 없어야 할 정보가 학습에 새어 성능이 부풀려지는 치명적 실수 | [튜닝·교차검증](06-hyperparameter-tuning-cv.md) | 파이프라인, 검증 |
| 스태킹(Stacking) | 여러 베이스 모델의 예측을 메타 모델이 다시 학습해 결합하는 앙상블 | [스태킹 앙상블](07-stacking-ensemble.md) | 메타 모델, 앙상블 |
| 메타 모델(Meta Model) | 베이스 모델들의 예측을 입력받아 최종 결정을 학습하는 2차 모델 | [스태킹 앙상블](07-stacking-ensemble.md) | 스태킹, 베이스 모델 |
| out-of-fold 예측 | 학습에 쓰지 않은 폴드로 만든 예측. 스태킹의 누수를 막는 핵심 | [스태킹 앙상블](07-stacking-ensemble.md) | 교차검증, 데이터 누수 |
