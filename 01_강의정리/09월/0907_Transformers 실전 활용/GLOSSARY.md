# 용어집

| 용어 | 설명 |
|---|---|
| Backbone | 일반적인 입력 표현을 만드는 사전학습 모델 본체 |
| Task head | 분류·회귀 등 특정 출력으로 변환하는 계층 |
| Pooling | 토큰 표현을 문장·샘플 수준 표현으로 모으는 처리 |
| Logit | 확률 변환 전 모델 출력 점수 |
| Multitask learning | 하나의 모델이 여러 학습 목표를 함께 최적화하는 방식 |
| Ensemble | 여러 모델의 예측을 결합하는 방식 |
| Negative transfer | 한 과업 학습이 다른 과업 성능을 떨어뜨리는 현상 |
| Dataset | 하나의 split을 나타내는 열 기반 데이터 객체 |
| DatasetDict | 여러 split의 Dataset을 묶은 객체 |
| IterableDataset | 데이터를 순차적으로 읽는 스트리밍 데이터 객체 |
| Arrow | 열 기반 메모리·파일 데이터 포맷 |
| Memory mapping | 파일 일부를 가상 메모리처럼 접근하는 방식 |
| Features | Dataset 열 이름과 자료형·구조를 나타내는 스키마 |
| Data collator | 샘플 목록을 padding해 학습 batch tensor로 만드는 함수 |
| Metric | 예측 품질을 측정하는 평가 기준 |
| Precision | 양성 예측 중 실제 양성의 비율 |
| Recall | 실제 양성 중 찾아낸 비율 |
| F1 | Precision과 Recall의 조화 평균 |
| Trainer | Transformers 모델의 학습·평가 루프를 관리하는 API |
| TrainingArguments | batch, 학습률, 평가·저장 등 학습 설정 묶음 |
| Gradient accumulation | 여러 작은 batch의 gradient를 누적해 갱신하는 기법 |
| Checkpoint | 모델과 학습 재개 상태의 중간 저장본 |
| Fine-tuning | 사전학습 모델을 특정 데이터·과업에 맞게 추가 학습하는 과정 |
| Mixed precision | 여러 수치 정밀도를 사용해 학습 자원을 줄이는 기법 |

## 함께 보기

- [사전학습 백본에 커스텀 헤드 추가하기](01-custom-model-heads.md)
- [Evaluate로 평가 파이프라인 만들기](05-evaluate-metrics-pipeline.md)
- [Trainer API 학습 흐름](06-trainer-api-workflow.md)
