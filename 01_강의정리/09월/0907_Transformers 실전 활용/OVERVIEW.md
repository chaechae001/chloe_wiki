# Transformers 실전 활용

사전학습 백본에 커스텀 헤드를 추가하는 것부터 멀티태스크·앙상블, Datasets 전처리, Evaluate 평가, Trainer 파인튜닝과 재현 가능한 운영까지 전체 실전 흐름을 학습합니다.

## 학습 목표

- 백본과 과업 헤드의 입력·출력 계약을 설계합니다.
- 멀티태스크 loss와 앙상블 결합 기준을 비교합니다.
- Dataset·DatasetDict·IterableDataset과 핵심 연산을 구분합니다.
- 평가 지표를 오류 비용과 데이터 분포에 맞게 선택합니다.
- Trainer 학습·평가·checkpoint 흐름을 재현 가능하게 관리합니다.

## 추천 학습 순서

1. [사전학습 백본에 커스텀 헤드 추가하기](01-custom-model-heads.md)
2. [멀티태스크 학습과 앙상블](02-multitask-and-ensembles.md)
3. [Datasets 구조와 핵심 연산](03-datasets-core-operations.md)
4. [커스텀 데이터셋과 토큰화 전처리](04-custom-dataset-preprocessing.md)
5. [Evaluate로 평가 파이프라인 만들기](05-evaluate-metrics-pipeline.md)
6. [Trainer API 학습 흐름](06-trainer-api-workflow.md)
7. [파인튜닝 운영과 재현성](07-finetuning-operations.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
과업·기준선 정의 → 데이터 스키마·split 검증 → batch 토큰화
→ 백본·헤드 구성 → Trainer 학습·checkpoint → Evaluate 검증
→ best 모델 선택 → test 평가 → 모델·토크나이저·실험 기록 저장
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| 백본 위 분류·회귀 head | 01 커스텀 헤드 |
| 여러 목표·모델 결합 | 02 멀티태스크·앙상블 |
| Arrow, map, filter, split | 03 Datasets 연산 |
| 파일·토큰화·동적 padding | 04 커스텀 전처리 |
| accuracy, F1, 평가 콜백 | 05 Evaluate |
| TrainingArguments와 학습 루프 | 06 Trainer |
| 기준선, checkpoint, 재현성 | 07 파인튜닝 운영 |

## 최종 점검

- [ ] 모델 입력·출력·loss shape를 검증한다.
- [ ] 데이터 split 누수와 전처리 손실을 확인한다.
- [ ] 오류 비용에 맞는 복수 지표를 사용한다.
- [ ] 평가·저장·best model 조건을 일치시킨다.
- [ ] 데이터·코드·환경·설정·checkpoint를 함께 기록한다.
