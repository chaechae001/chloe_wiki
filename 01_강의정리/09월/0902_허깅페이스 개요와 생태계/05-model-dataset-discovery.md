# Model Hub와 Dataset Hub 탐색

검색 결과의 인기 순위만 따라가면 과업·언어·라이선스가 맞지 않는 자원을 고르기 쉽습니다. 필터는 후보를 줄이는 도구이고, 최종 선택은 카드와 실제 샘플 검증으로 결정합니다.

**핵심 키워드:** task, language, license, dataset card, Data Studio

## 모델 탐색 순서

1. 입력과 기대 출력을 과업으로 정의합니다.
2. 언어, 라이브러리, 라이선스와 크기로 후보를 좁힙니다.
3. 모델 카드에서 사용 목적과 제한을 확인합니다.
4. 가중치 형식과 필요한 자원을 계산합니다.
5. 자체 검증 세트로 정확도·지연·안전을 평가합니다.

## 데이터셋 탐색 순서

Dataset Card에서 데이터 출처, 수집 방법, 스키마, split, 라이선스와 알려진 편향을 확인합니다. Data Studio 같은 미리보기는 구조와 샘플을 빠르게 이해하는 데 도움을 주지만 전체 품질 검사를 대신하지 않습니다.

```python
from datasets import load_dataset

sample = load_dataset(
    "organization/reviewed-dataset",
    split="train[:100]",
    revision="reviewed-commit-id",
)
print(sample.features)
print(sample.num_rows)
```

먼저 작은 범위로 스키마와 값 분포를 확인하고, 민감 정보와 사용 목적 적합성을 점검한 뒤 확대합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| 태그 vs 검증 근거 | 검색용 메타데이터 vs 실제 성능을 뒷받침하는 실험 |
| Dataset Card vs Data Studio | 출처·조건 설명 vs 데이터 파일 시각적 탐색 |
| train split vs test split | 모델 학습용 데이터 vs 최종 일반화 평가용 데이터 |

## 직접 해보기

1. 한국어 감성 분류 모델의 검색 필터를 정하세요.
2. 데이터 카드에서 반드시 확인할 항목을 나열하세요.
3. 작은 샘플을 먼저 읽는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 과업, 한국어, 라이선스, 프레임워크와 실행 가능한 크기를 봅니다.
2. 출처, 수집·전처리, 스키마, split, 라이선스, 편향과 제한을 확인합니다.
3. 비용을 줄이며 구조 오류, 민감 정보와 품질 문제를 조기에 찾을 수 있습니다.

</details>

## 연결되는 개념

- 이전: [Hub 저장소와 버전 관리](04-hub-repositories.md)
- 다음: [Spaces로 모델 데모 만들기](06-spaces-and-ecosystem.md)

## 셀프 체크

- [ ] 과업 중심으로 후보를 검색한다.
- [ ] 모델 카드의 제한을 확인한다.
- [ ] 데이터 출처와 라이선스를 확인한다.
- [ ] split의 목적을 구분한다.
- [ ] 자체 샘플로 품질을 검증한다.

### 복습 질문 및 답변

**Q1. 다운로드 수가 많으면 내 과업에도 좋은 모델인가요?**

<details>
<summary>답</summary>

아닙니다. 인기도는 과업·언어·도메인 적합성과 같은 지표가 아닙니다.

</details>

**Q2. Dataset Card가 없으면 어떤 문제가 있나요?**

<details>
<summary>답</summary>

출처, 사용 조건과 한계를 판단하기 어려워 재현성과 법적·윤리적 검토가 약해집니다.

</details>

**Q3. test split으로 반복 튜닝하면 왜 안 되나요?**

<details>
<summary>답</summary>

평가 데이터에 의사결정이 맞춰져 최종 성능이 실제보다 낙관적으로 보일 수 있습니다.

</details>

## 한 줄 정리

> Hub 탐색은 태그로 후보를 줄이고 카드·샘플·자체 평가로 선택을 검증하는 과정입니다.
