# OVERVIEW

## 학습 목표

이차전지 양극재 물성 데이터를 이해하고, 전처리부터 전통 머신러닝과 딥러닝 모델 비교, 일반화 성능 개선까지 하나의 분류 프로젝트로 수행합니다.

## 학습 로드맵

1. [배터리 양극재 프로젝트와 데이터셋](01-battery-cathode-project-and-dataset.md)
2. [데이터 탐색, 결측치와 이상치](02-eda-missing-values-and-outliers.md)
3. [인코딩, 스케일링과 피처 엔지니어링](03-encoding-scaling-and-feature-engineering.md)
4. [선형 회귀와 로지스틱 회귀](04-linear-and-logistic-models.md)
5. [PCA와 전통 분류 모델](05-pca-and-classical-classifiers.md)
6. [MLP 분류 모델과 최적화](06-mlp-classification-and-optimization.md)
7. [일반화와 딥러닝 성능 개선](07-generalization-and-deep-learning-improvements.md)
8. [양극재 결정구조 분류 전체 파이프라인](08-end-to-end-cathode-classification-workflow.md)
9. [핵심 용어집](GLOSSARY.md)

## 단계별 산출물

| 단계 | 산출물 |
|---|---|
| 문제 정의 | 목표값, 입력 피처, 평가 지표 |
| 탐색 | 자료형·결측·분포·클래스 균형 점검 |
| 전처리 | 누수 없는 인코딩·스케일링 파이프라인 |
| 기준 모델 | 로지스틱 회귀 교차검증 결과 |
| 모델 비교 | 전통 분류기와 MLP의 동일 조건 비교 |
| 개선 | 조기 종료·규제·드롭아웃 실험 |
| 최종 평가 | 테스트 지표, 혼동행렬, 오류 분석 |

## 권장 학습 방법

- 각 문서의 코드 예제를 직접 실행합니다.
- 전처리 단계마다 학습 데이터에서만 기준을 계산했는지 확인합니다.
- 모델을 바꿀 때 데이터 분할과 평가지표를 고정합니다.
- 각 문서 끝의 복습 질문을 먼저 답한 뒤 토글을 펼쳐 확인합니다.

## 핵심 원칙

> 좋은 성능은 복잡한 모델보다 정확한 문제 정의, 누수 없는 평가, 타당한 도메인 피처에서 시작합니다.
