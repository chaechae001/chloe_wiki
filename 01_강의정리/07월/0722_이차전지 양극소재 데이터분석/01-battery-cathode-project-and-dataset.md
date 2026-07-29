# 배터리 양극재 결정구조 분류 프로젝트와 데이터셋

## 한 줄 요약

양극재의 물성 데이터를 입력으로 받아 결정계(`Crystal System`)를 예측하는 다중 분류 문제를 정의하고, 데이터와 평가 절차를 먼저 이해합니다.

## 왜 양극재를 분석하나요?

- 이차전지는 양극재, 음극재, 분리막, 전해액으로 구성됩니다.
- 양극재는 배터리의 용량과 평균 전압에 큰 영향을 줍니다.
- 대표 양극재에는 NCA, NCM, LFP, LCO가 있으며 조성에 따라 성능과 안정성이 달라집니다.
- 재료 탐색에 AI를 적용하면 많은 후보를 빠르게 선별할 수 있습니다.

## Materials Project와 실습 데이터

Materials Project는 계산 재료과학 데이터를 제공하는 공개 데이터베이스입니다. 실습 데이터는 리튬 화합물의 물성을 정리한 339개 행, 11개 항목으로 구성됩니다.

| 구분 | 예시 | 의미 |
|---|---|---|
| 식별 정보 | `Materials Id`, `Formula` | 재료 ID와 화학식 |
| 구조 정보 | `Spacegroup`, `Crystal System` | 공간군과 결정계 |
| 에너지 | `Formation Energy`, `E Above Hull` | 형성 에너지와 안정성 지표 |
| 전자·기하 특성 | `Band Gap`, `Nsites`, `Density`, `Volume` | 밴드갭, 원자 수, 밀도, 부피 |
| 부가 정보 | `Has Bandstructure` | 밴드 구조 보유 여부 |

목표값 `Crystal System`은 monoclinic, orthorhombic, triclinic의 세 클래스입니다.

## 문제 정의

```python
target_col = "Crystal System"
y = df[target_col]
X = df.drop(columns=target_col)
```

이는 연속값 예측이 아니라 **다중 클래스 분류**입니다. 따라서 최종 성능은 정확도만 보지 말고 클래스별 정밀도, 재현율, F1 점수도 함께 확인해야 합니다.

## 학습·검증·테스트의 역할

- 학습 데이터: 모델이 패턴을 학습합니다.
- 검증 데이터: 모델 선택과 과적합 판단에 사용합니다.
- 테스트 데이터: 모든 선택이 끝난 모델의 일반화 성능을 한 번 평가합니다.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)
```

`stratify=y`는 작은 데이터에서 클래스 비율이 한쪽으로 치우치는 위험을 줄입니다.

## 실무 체크리스트

- 목표값이 입력 피처에 남아 있지 않은가?
- 재료 ID처럼 단순 식별자인 열을 학습에 넣지 않았는가?
- 전처리 기준을 전체 데이터가 아니라 학습 데이터에서만 구했는가?
- 테스트 데이터를 모델 선택에 반복 사용하지 않았는가?

## 복습 질문 및 답변

**Q1. 이 프로젝트가 회귀가 아니라 분류인 이유는 무엇인가요?**

<details>
<summary>답</summary>
	예측 대상이 연속적인 수치가 아니라 monoclinic, orthorhombic, triclinic처럼 구분된 범주이기 때문입니다.
</details>

**Q2. `Materials Id`를 피처에서 제거하는 이유는 무엇인가요?**

<details>
<summary>답</summary>
	재료를 구별하기 위한 식별자일 뿐 일반화 가능한 물성을 직접 나타내지 않아 모델이 우연한 패턴을 외울 수 있기 때문입니다.
</details>

**Q3. 테스트 데이터는 언제 사용해야 하나요?**

<details>
<summary>답</summary>
	전처리 방법과 모델, 하이퍼파라미터 선택이 모두 끝난 뒤 최종 일반화 성능을 확인할 때 사용합니다.
</details>

## 다음 학습

[데이터 탐색과 전처리](02-eda-missing-values-and-outliers.md)에서 결측치와 이상치를 다룹니다. 전체 순서는 [OVERVIEW](OVERVIEW.md)에서 확인할 수 있습니다.
