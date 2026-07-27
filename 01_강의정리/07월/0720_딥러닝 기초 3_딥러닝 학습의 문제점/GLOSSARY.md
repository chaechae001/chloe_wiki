# 용어집

이번 회차의 핵심 용어를 문제 진단, 학습 단위, 옵티마이저, 신호 전파, 일반화와 정규화로 나누어 정리했습니다. 각 용어의 관련 글을 따라가면 정의에서 실험 해석까지 이어서 복습할 수 있습니다.

## 학습 문제 진단

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 최적화 실패 | 훈련 데이터에서도 손실이 충분히 줄지 않아 파라미터가 좋은 지점에 도달하지 못한 상태 | [01](01-deep-learning-training-failure-modes.md) | 학습률, 기울기 |
| 기울기 소실 | 역전파 신호가 층을 거슬러 갈수록 매우 작아져 입력 쪽 가중치가 거의 갱신되지 않는 현상 | [04](04-vanishing-gradient-and-activation-functions.md) | 포화, ReLU |
| 초기화 문제 | 시작 가중치의 크기가 맞지 않아 활성값이나 기울기가 깊은 층에서 사라지거나 불안정해지는 문제 | [05](05-weight-initialization-and-signal-propagation.md) | Xavier, He |
| 과적합 | 훈련 데이터에는 잘 맞지만 보지 않은 데이터의 성능은 충분히 좋아지지 않는 상태 | [06](06-overfitting-and-regularization.md) | 일반화, 규제 |
| 일반화 격차 | 훈련 성능과 검증 성능 사이의 차이. 격차만이 아니라 검증 성능 자체도 함께 봐야 함 | [06](06-overfitting-and-regularization.md) | 검증 데이터, 과적합 |
| 진단 순서 | 데이터·손실을 확인한 뒤 학습 곡선, 스텝, 기울기, 활성값, 일반화 간격을 차례로 점검하는 흐름 | [01](01-deep-learning-training-failure-modes.md) | 기준선, 단일 변수 실험 |

## 배치와 학습량

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| Full-batch | 전체 훈련 데이터를 한꺼번에 사용해 기울기와 한 번의 업데이트를 계산하는 방식 | [02](02-mini-batch-and-optimization-basics.md) | Mini-batch, Step |
| Mini-batch | 전체 데이터의 작은 묶음마다 기울기를 추정하고 파라미터를 갱신하는 방식 | [02](02-mini-batch-and-optimization-basics.md) | 배치 크기, 셔플 |
| Epoch | 모델이 전체 훈련 데이터를 한 번 사용한 단위. 배치 크기가 다르면 같은 에포크의 스텝 수도 달라짐 | [02](02-mini-batch-and-optimization-basics.md) | Step, 데이터 노출량 |
| Step | 배치 하나로 순전파·역전파·파라미터 갱신을 마친 한 번의 학습 단계 | [02](02-mini-batch-and-optimization-basics.md) | Epoch, 업데이트 수 |
| 학습률 | 한 스텝에서 기울기를 얼마나 크게 반영할지 정하는 기본 이동 크기 | [02](02-mini-batch-and-optimization-basics.md) | 수렴, 발산 |
| 벽시계 시간 | 학습 시작부터 종료까지 실제로 흐른 시간. 에포크나 스텝 수와 다른 비용 기준 | [02](02-mini-batch-and-optimization-basics.md) | 처리량, 시간 예산 |
| 공정 비교 | 비교 질문에 맞춰 초기값, 데이터 순서, 업데이트 수, 시간 예산과 평가 시점 등을 통제하는 실험 원칙 | [02](02-mini-batch-and-optimization-basics.md) | 반복 실험, 난수 시드 |

## 옵티마이저

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| SGD | 현재 미니배치의 기울기 반대 방향으로 가중치를 이동하는 기본 옵티마이저 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | 학습률, Mini-batch |
| Momentum | 과거 이동 방향을 누적해 반복되는 방향은 강화하고 좌우 진동은 줄이는 갱신 방식 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | 속도, 관성 계수 |
| AdaGrad | 기울기 제곱을 계속 누적해 파라미터마다 유효 학습률을 조절하는 방법 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | 희소 특성, 누적합 |
| RMSProp | 최근 기울기 제곱에 더 큰 비중을 두는 이동 평균으로 보폭을 조절하는 방법 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | AdaGrad, 감쇠율 |
| Adam | 기울기의 방향 평균과 제곱 평균을 함께 사용해 이동 방향과 보폭을 조절하는 옵티마이저 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | Momentum, RMSProp |
| 유효 학습률 | 기본 학습률이 기울기 이력에 따라 파라미터별로 조정된 실제 이동 크기 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | 적응형 옵티마이저 |
| 조건부 실험 해석 | 한 번의 손실 수치를 알고리즘의 절대 순위로 보지 않고 해당 설정에서의 관찰로 제한하는 태도 | [03](03-sgd-momentum-and-adaptive-optimizers.md) | 반복 평균, 검증 성능 |

## 기울기 흐름과 초기화

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 역전파 | 출력 쪽 손실에서 시작해 연쇄법칙으로 각 층의 기울기를 거꾸로 계산하는 과정 | [04](04-vanishing-gradient-and-activation-functions.md) | 기울기 소실, 연쇄법칙 |
| Sigmoid | 입력을 0과 1 사이로 바꾸지만 양 끝에서 도함수가 작아져 깊은 은닉층에서는 포화가 생길 수 있는 함수 | [04](04-vanishing-gradient-and-activation-functions.md) | 포화, 기울기 소실 |
| Tanh | 입력을 -1과 1 사이로 바꾸고 0을 중심으로 하지만 큰 절댓값에서는 역시 포화되는 함수 | [04](04-vanishing-gradient-and-activation-functions.md) | Sigmoid, Xavier |
| ReLU | 음수는 0, 양수는 그대로 통과시켜 양수 구간의 기울기를 유지하는 활성화 함수 | [04](04-vanishing-gradient-and-activation-functions.md) | He 초기화, Dead ReLU |
| Dead ReLU | 입력이 계속 음수 영역에 머물러 ReLU의 출력과 기울기가 0이 되고 뉴런이 갱신되기 어려운 상태 | [04](04-vanishing-gradient-and-activation-functions.md) | 학습률, 편향 |
| `fan_in`·`fan_out` | 한 뉴런으로 들어오는 연결 수와 한 층에서 다음 층으로 나가는 연결 수. 초기 가중치 스케일의 기준 | [05](05-weight-initialization-and-signal-propagation.md) | 분산, 신호 전파 |
| Xavier 초기화 | 입력·출력 연결 수를 고려해 신호 크기를 조절하며 선형·Tanh 계열에 자주 쓰이는 초기화 | [05](05-weight-initialization-and-signal-propagation.md) | Tanh, 분산 |
| He 초기화 | ReLU가 일부 출력을 0으로 만드는 특성을 고려해 비교적 큰 분산을 사용하는 초기화 | [05](05-weight-initialization-and-signal-propagation.md) | ReLU, `fan_in` |

## 일반화와 정규화

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| L1 정규화 | 손실에 가중치 절댓값의 합을 더해 일부 가중치가 0에 가까워지도록 유도하는 규제 | [06](06-overfitting-and-regularization.md) | 희소성, 규제 강도 |
| L2 정규화 | 손실에 가중치 제곱합을 더해 지나치게 큰 가중치를 부드럽게 억제하는 규제 | [06](06-overfitting-and-regularization.md) | 가중치 감쇠, 일반화 |
| 검증 데이터 | 모델과 하이퍼파라미터를 선택하는 동안 반복해서 성능을 확인하는 데이터 | [06](06-overfitting-and-regularization.md) | 테스트 데이터, 모델 선택 |
| 테스트 데이터 | 모든 선택이 끝난 뒤 최종 일반화 성능을 평가하기 위해 남겨 두는 데이터 | [06](06-overfitting-and-regularization.md) | 검증 데이터, 데이터 누수 |
| Dropout | 학습 중 일부 활성값을 무작위로 0으로 만들어 특정 경로에 대한 과도한 의존을 줄이는 방법 | [07](07-dropout-and-batch-normalization.md) | 학습 모드, 공동 적응 |
| Inverted Dropout | 학습할 때 살아남은 활성값을 보정해 추론 시 별도의 확률 곱셈 없이 전체 출력을 사용하는 구현 | [07](07-dropout-and-batch-normalization.md) | Dropout 비율, 추론 모드 |
| Batch Normalization | 학습 중 배치 통계로 중간 활성값을 정규화하고 학습 가능한 크기·이동 값을 적용하는 층 | [07](07-dropout-and-batch-normalization.md) | 이동 평균, $\gamma$·$\beta$ |
| 학습 모드·추론 모드 | 학습 때는 Dropout 마스크와 현재 배치 통계를 쓰고, 추론 때는 Dropout을 끄고 누적 통계를 쓰는 동작 구분 | [07](07-dropout-and-batch-normalization.md) | 결정론적 출력, 이동 통계 |
