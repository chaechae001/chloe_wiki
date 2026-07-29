# 양극재 결정구조 분류 전체 파이프라인

## 한 줄 요약

문제 정의부터 전처리, 모델 비교, 딥러닝 개선, 최종 평가까지 재현 가능한 하나의 흐름으로 연결합니다.

## 1. 목표와 평가 기준 고정

- 입력: 양극재 물성 피처
- 목표: `Crystal System` 다중 분류
- 기본 지표: macro F1, 정확도, 혼동행렬
- 보조 지표: 클래스별 재현율, 다중 클래스 ROC-AUC

## 2. 데이터 분리부터 수행

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

교차검증이나 검증 세트는 학습 데이터 내부에서 구성합니다.

## 3. 전처리를 파이프라인에 묶기

```python
from sklearn.pipeline import Pipeline

baseline = Pipeline([
    ("preprocess", preprocess),
    ("model", LogisticRegression(max_iter=2000)),
])
baseline.fit(X_train, y_train)
```

파이프라인은 결측값 대체, 인코딩, 표준화가 검증 폴드마다 학습 데이터에만 맞춰지도록 합니다.

## 4. 기준 모델과 후보 모델 비교

1. 로지스틱 회귀로 기준 성능을 만듭니다.
2. 결정 트리, KNN, 랜덤 포레스트, XGBoost를 같은 교차검증으로 비교합니다.
3. PCA 사용 여부를 별도 실험으로 기록합니다.
4. MLP는 데이터 규모와 기준 모델 성능을 확인한 뒤 비교합니다.

## 5. 평가 보고서

```python
from sklearn.metrics import classification_report, confusion_matrix

pred = best_model.predict(X_test)
print(classification_report(y_test, pred))
print(confusion_matrix(y_test, pred))
```

전체 정확도뿐 아니라 어떤 결정계를 어떤 클래스로 혼동하는지 분석합니다.

## 6. 실험 기록

| 항목 | 기록 내용 |
|---|---|
| 데이터 | 버전, 행 수, 피처 목록 |
| 분할 | seed, 비율, stratify 여부 |
| 전처리 | 결측, 인코딩, 스케일링 기준 |
| 모델 | 알고리즘과 하이퍼파라미터 |
| 결과 | 검증 평균·편차, 테스트 지표 |
| 해석 | 오류 유형과 다음 실험 |

## 7. 개선 우선순위

- 데이터 누수와 평가 설계를 먼저 점검합니다.
- 도메인 피처를 개선합니다.
- 교차검증으로 모델을 비교합니다.
- 그 다음 하이퍼파라미터와 딥러닝 규제를 조정합니다.
- 작은 테스트 점수 차이 하나만으로 모델을 선택하지 않습니다.

## 복습 질문 및 답변

**Q1. 전처리를 파이프라인에 넣는 가장 큰 이유는 무엇인가요?**

<details>
<summary>답</summary>
	각 검증 단계에서 전처리가 학습 데이터에만 맞춰져 데이터 누수를 막고, 학습과 추론에 동일한 변환을 적용하기 위해서입니다.
</details>

**Q2. 기준 모델을 먼저 만드는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	복잡한 모델의 개선 폭과 비용이 실제로 의미 있는지 판단할 비교 기준을 확보하기 위해서입니다.
</details>

**Q3. 정확도가 높아도 혼동행렬을 확인해야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	특정 소수 클래스를 거의 맞히지 못하거나 특정 결정계 쌍을 반복해서 혼동하는 문제를 전체 정확도가 숨길 수 있기 때문입니다.
</details>

## 마무리

[OVERVIEW](OVERVIEW.md)에서 전체 학습 순서를 다시 보고, [GLOSSARY](GLOSSARY.md)에서 핵심 용어를 복습하세요.
