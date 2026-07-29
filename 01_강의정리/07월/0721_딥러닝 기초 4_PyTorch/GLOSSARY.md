# PyTorch 핵심 용어집

## 텐서와 연산

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| Tensor | 여러 차원의 숫자 배열이자 PyTorch 연산의 기본 단위 | [1편](01-pytorch-tensors-and-operations.md) |
| shape | 텐서의 각 축 크기 | [1편](01-pytorch-tensors-and-operations.md) |
| dtype | 텐서 원소의 자료형 | [1편](01-pytorch-tensors-and-operations.md) |
| device | 텐서와 모델이 계산되는 CPU 또는 GPU | [1편](01-pytorch-tensors-and-operations.md) |
| logits | 출력 활성화 전 클래스별 원시 점수 | [4편](04-loss-optimizers-and-training-loop.md) |

## 모델과 자동미분

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| autograd | 연산 기록으로 기울기를 자동 계산하는 기능 | [2편](02-autograd-modules-and-model-modes.md) |
| gradient | 손실을 줄이는 파라미터 변화 방향과 크기 | [2편](02-autograd-modules-and-model-modes.md) |
| `nn.Module` | 신경망 계층과 순전파를 묶는 기본 클래스 | [2편](02-autograd-modules-and-model-modes.md) |
| `forward()` | 입력이 모델을 통과하는 순서를 정의하는 메서드 | [2편](02-autograd-modules-and-model-modes.md) |
| parameter | 학습 과정에서 갱신되는 가중치와 편향 | [2편](02-autograd-modules-and-model-modes.md) |

## 데이터 파이프라인

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| Dataset | 개별 입력과 라벨을 제공하는 규칙 또는 데이터 묶음 | [3편](03-datasets-dataloaders-and-splits.md) |
| DataLoader | Dataset을 미니배치로 묶어 반복하는 도구 | [3편](03-datasets-dataloaders-and-splits.md) |
| batch | 한 번의 순전파와 갱신에 사용하는 샘플 묶음 | [3편](03-datasets-dataloaders-and-splits.md) |
| epoch | 전체 학습 데이터를 한 번 순회한 단위 | [4편](04-loss-optimizers-and-training-loop.md) |
| shuffle | 매 에포크 데이터 순서를 섞는 설정 | [3편](03-datasets-dataloaders-and-splits.md) |

## 학습과 평가

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| loss | 예측과 정답의 차이를 나타내는 최적화 대상 값 | [4편](04-loss-optimizers-and-training-loop.md) |
| optimizer | 기울기를 사용해 파라미터를 갱신하는 알고리즘 | [4편](04-loss-optimizers-and-training-loop.md) |
| learning rate | 한 번의 갱신 크기를 조절하는 값 | [4편](04-loss-optimizers-and-training-loop.md) |
| accuracy | 전체 샘플 중 맞게 예측한 비율 | [5편](05-binary-classification-workshop.md) |
| validation | 모델 설정과 학습 시점을 선택하기 위한 평가 | [7편](07-evaluation-and-improvement.md) |
| test | 선택이 끝난 모델의 최종 일반화 성능 평가 | [7편](07-evaluation-and-improvement.md) |
| overfitting | 학습 데이터에는 좋지만 새 데이터 성능이 낮은 상태 | [7편](07-evaluation-and-improvement.md) |

## 빠른 구분

| 헷갈리는 개념 | 핵심 차이 |
|---|---|
| Dataset vs DataLoader | 샘플 제공 vs 배치 반복 |
| `train()` vs `eval()` | 학습 모드 vs 평가 모드 |
| `backward()` vs `step()` | 기울기 계산 vs 파라미터 갱신 |
| logits vs probability | 원시 점수 vs 확률 해석값 |
| validation vs test | 설정 선택 vs 최종 평가 |
| loss vs accuracy | 연속 오류값 vs 맞힌 비율 |

전체 연결은 [평가와 성능 개선](07-evaluation-and-improvement.md)에서 확인할 수 있습니다.
