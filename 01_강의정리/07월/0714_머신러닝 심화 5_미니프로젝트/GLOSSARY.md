# 용어집

이번 회차에 등장한 핵심 용어를 쉬운 말로 정리했습니다. 주제별로 나눠 두었으니, 학습 중 막히는 용어를 빠르게 찾아보세요.

## 문제 정의와 평가 설계

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 종속변수·독립변수 | 맞혀야 할 정답 열이 종속변수, 정답을 맞히려고 모델이 보는 재료 열이 독립변수. 식별자와 정답 자체는 재료에서 반드시 뺌 | [1편](01-miniproject-overview-and-problem-framing.md) | 문제 정의, 데이터 누수 |
| 회귀 vs 분류 | 종속변수가 연속된 숫자면 회귀, 정해진 범주면 분류. 여기서 평가지표까지 함께 결정됨 | [1편](01-miniproject-overview-and-problem-framing.md) | 구간화, 평가지표 |
| 구간화(Binning) | 연속값을 기준선으로 잘라 범주로 바꾸는 것. 회귀를 분류로 바꿀 수 있지만 금액의 세밀함은 사라짐 | [3편](03-travel-expense-regression-and-classification.md) | 회귀 vs 분류, 중앙값 |

## 정형 데이터 전처리

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| EDA(탐색적 데이터 분석) | 데이터를 구경하는 단계가 아니라, 뒤이을 전처리 판단에 근거를 대기 위한 관찰 | [2편](02-travel-expense-data-preprocessing.md) | 결측치, 이상치 |
| 다중 라벨 이진 인코딩 | 한 칸에 여러 값이 들어 있는 컬럼을 값마다 열로 펼쳐 0/1로 표시하는 방법 | [2편](02-travel-expense-data-preprocessing.md) | 원핫 인코딩, 파생변수 |
| IQR 이상치 제거 | 사분위 범위를 기준으로 극단적으로 동떨어진 값을 걸러내는 방법. 지출 데이터처럼 꼬리가 긴 분포에 필수 | [2편](02-travel-expense-data-preprocessing.md) | 이상치, 박스플롯 |
| 병합과 행의 단위 | 표를 합치는 일의 본질은 키 맞추기가 아니라 "1행이 무엇을 의미하는가"를 통일하는 것 | [2편](02-travel-expense-data-preprocessing.md) | 집계, 조인 키 |

## 회귀 모델링과 평가

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| RMSE | 오차를 제곱해 평균 낸 뒤 제곱근을 씌운 값. 오차의 "크기"를 원래 단위로 보여 줌 | [3편](03-travel-expense-regression-and-classification.md) | MAPE, 이상치 |
| MAPE | 오차가 실제값 대비 몇 퍼센트인지로 재는 지표. 금액 규모가 달라도 비교할 수 있음 | [3편](03-travel-expense-regression-and-classification.md) | RMSE |
| R²(결정계수) | 모델이 정답의 변동을 얼마나 설명하는지 나타내는 값. 변수를 늘리기만 해도 오름 | [3편](03-travel-expense-regression-and-classification.md) | Adjusted R² |
| Adjusted R² | 변수 개수에 벌점을 매긴 R². 쓸모없는 변수를 넣으면 오히려 떨어져 착시를 걷어냄 | [3편](03-travel-expense-regression-and-classification.md) | R², 변수 선택 |

## 텍스트 전처리와 벡터화

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 형태소 분석·토큰화 | 조사·어미가 달라붙는 한국어를 뜻을 가진 최소 단위로 잘라, 같은 뜻의 단어가 한 특성으로 모이게 하는 작업 | [4편](04-korean-text-preprocessing.md) | 명사 추출, 정제 |
| BoW(Bag of Words) | 문장을 "어떤 단어가 몇 번 나왔는가"로만 표현하는 방식. 순서 정보는 버림 | [4편](04-korean-text-preprocessing.md) | TF-IDF, 어휘 사전 |
| TF-IDF | "이 문서에 자주 나오면 올리고, 모든 문서에 나오면 내린다"를 곱해 단어의 변별력을 점수화 | [4편](04-korean-text-preprocessing.md) | BoW, 희소 행렬 |
| 어휘 사전(vocabulary) | 벡터의 열이 될 단어 목록. 사전 크기가 곧 벡터의 차원 수가 됨 | [4편](04-korean-text-preprocessing.md) | 희소 행렬, BoW |
| 희소 행렬(Sparse Matrix) | 값이 거의 다 0인 행렬을 0이 아닌 위치만 저장해 다루는 형식. 텍스트 벡터는 대부분 이 형태 | [4편](04-korean-text-preprocessing.md) | 어휘 사전, 고차원 |

## 분류 평가와 클래스 불균형

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 정확도(Accuracy) | 전체 중 맞힌 비율. 요약일 뿐이라 어떤 클래스를 통째로 놓쳤는지는 보이지 않음 | [5편](05-text-classification-and-evaluation.md) | F1, 클래스 불균형 |
| 정밀도·재현율·F1 | 정밀도는 "A라고 말한 것 중 실제 A였던 비율", 재현율은 "실제 A 중 A라고 맞힌 비율", F1은 둘의 조화평균 | [5편](05-text-classification-and-evaluation.md) | 혼동행렬, macro 평균 |
| macro / weighted 평균 | macro는 모든 클래스를 동등하게, weighted는 샘플 수에 비례해 평균. 둘의 차이가 곧 소수 클래스 성적표 | [5편](05-text-classification-and-evaluation.md) | 클래스 불균형, F1 |
| 혼동행렬 | 어느 클래스가 어느 클래스로 잘못 흘러갔는지 방향까지 보여 주는 표 | [5편](05-text-classification-and-evaluation.md) | 오분류, 재현율 |
| 클래스 불균형·언더샘플링 | 클래스별 데이터 수가 크게 차이 나는 상태와, 많은 쪽을 덜어내 균형을 맞추는 대응책 | [5편](05-text-classification-and-evaluation.md) | macro F1, 가중치 |

## 모델과 앙상블

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 선형 SVM | 클래스 사이에 여유를 두고 경계선을 긋는 모델. 고차원 희소 텍스트 벡터에서 특히 강함 | [5편](05-text-classification-and-evaluation.md) | 나이브 베이즈, KNN |
| 나이브 베이즈 | 단어별 등장 확률을 곱해 분류하는 확률 모델. 빠르고 텍스트 데이터에 잘 맞음 | [5편](05-text-classification-and-evaluation.md) | 선형 SVM, BoW |
| 하드 보팅 | 여러 모델의 최종 예측을 모아 다수결로 정하는 앙상블. 모델들이 같은 곳에서 틀리면 이득이 없음 | [5편](05-text-classification-and-evaluation.md) | 앙상블, 다양성 |
| 랜덤포레스트 | 서로 다른 샘플·피처로 만든 트리 여러 개가 투표하는 방식(배깅). 단일 트리보다 안정적이며 그 자체가 앙상블 | [6편](06-sensor-multiclass-classification.md) | 의사결정트리, 부스팅 |
| 부스팅 | 앞 트리가 틀린 부분을 다음 트리가 집중해 보완하는 순차 학습. 겹쳐 있는 소수 클래스에 강함 | [6편](06-sensor-multiclass-classification.md) | 랜덤포레스트, 학습률 |
| 스케일링·라벨 인코딩 | 열마다 제각각인 값의 범위를 같은 잣대로 맞추는 것과, 문자열 정답을 정수 번호로 바꾸는 것 | [6편](06-sensor-multiclass-classification.md) | MinMax, 전처리 |

## 교차검증과 튜닝

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| K-Fold 교차검증 | 데이터를 K조각으로 나눠 모두 한 번씩 검증셋이 되게 하고, 그 평균으로 성능을 판단하는 방법 | [7편](07-cross-validation-and-hyperparameter-tuning.md) | StratifiedKFold |
| StratifiedKFold | 각 폴드의 클래스 비율을 원본과 같게 유지하는 분할. 불균형 데이터에서는 사실상 필수 | [7편](07-cross-validation-and-hyperparameter-tuning.md) | K-Fold, 클래스 불균형 |
| L1·L2 정규화 | 가중치가 커지는 데 벌점을 매겨 과적합을 억제하는 장치. 효과는 경계·소수 클래스에서 먼저 나타남 | [7편](07-cross-validation-and-hyperparameter-tuning.md) | 과적합, 하이퍼파라미터 |
| 하이퍼파라미터 | 학습으로 배우는 값이 아니라 사람이 미리 정하는 설정값(트리 깊이, 학습률 등) | [7편](07-cross-validation-and-hyperparameter-tuning.md) | 자동 탐색, 학습률 |
| 목적함수·탐색공간·시행 횟수 | 튜닝 자동화에 필요한 세 가지 — 조합의 점수를 매기는 함수, 후보의 범위, 실제로 실험할 횟수 | [7편](07-cross-validation-and-hyperparameter-tuning.md) | 하이퍼파라미터, 교차검증 |
