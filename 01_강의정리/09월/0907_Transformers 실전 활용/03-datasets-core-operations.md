# Datasets 구조와 핵심 연산

Hugging Face Datasets는 데이터를 열 단위 스키마와 split으로 관리하고, 캐시 가능한 변환을 통해 반복 가능한 전처리 흐름을 만듭니다.

**핵심 키워드:** Dataset, DatasetDict, IterableDataset, Arrow, map

## 데이터 구조

| 구조 | 용도 |
|---|---|
| `Dataset` | 하나의 표 형태 데이터 집합 |
| `DatasetDict` | train·validation·test 같은 split 묶음 |
| `IterableDataset` | 순차 스트리밍·대규모 데이터 처리 |

Arrow 기반 열 저장과 메모리 매핑은 데이터를 매번 Python 객체로 전부 복사하지 않고 효율적으로 읽는 데 도움을 줍니다. “zero-copy”는 모든 연산이 항상 복사 0회라는 뜻이 아니라 가능한 경로에서 불필요한 복사를 줄이는 특성으로 이해합니다.

```python
from datasets import Dataset

ds = Dataset.from_dict({
    "text": ["clear guide", "", "useful example"],
    "label": [1, 0, 1],
})

clean = (
    ds.filter(lambda row: bool(row["text"].strip()))
      .map(lambda row: {"length": len(row["text"])})
      .sort("length")
)
print(clean.column_names, len(clean))
```

## 연산 구분

`map`은 행·batch를 변환하고, `filter`는 조건에 맞는 행을 남깁니다. `select`는 인덱스로 일부 행을 고르고, `sort`는 열 기준 순서를 바꿉니다. 원본 객체를 제자리 수정한다고 가정하지 말고 반환된 Dataset을 이어서 사용합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| map vs filter | 값·열 변환 vs 행 유지 여부 결정 |
| Dataset vs DatasetDict | 단일 split vs 여러 split 컨테이너 |
| Dataset vs IterableDataset | 인덱스 접근 중심 vs 순차 스트림 중심 |

## 직접 해보기

1. 빈 텍스트 제거와 길이 열 생성을 구현하세요.
2. train과 validation의 전처리를 동일하게 적용하세요.
3. 스트리밍에서 임의 인덱스 접근이 어려운 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. `filter` 뒤 `map`을 사용합니다.
2. 같은 변환 함수를 `DatasetDict.map()`에 적용하고 split별 라벨 분포를 다시 확인합니다.
3. 데이터가 필요할 때 순서대로 도착해 전체 위치 정보가 메모리에 없을 수 있기 때문입니다.

</details>

## 연결되는 개념

- 이전: [멀티태스크 학습과 앙상블](02-multitask-and-ensembles.md)
- 다음: [커스텀 데이터셋과 토큰화 전처리](04-custom-dataset-preprocessing.md)

## 셀프 체크

- [ ] 세 데이터 구조를 구분한다.
- [ ] Arrow와 mmap의 목적을 설명한다.
- [ ] 핵심 네 연산을 선택한다.
- [ ] 반환 Dataset을 이어서 사용한다.
- [ ] split별 분포를 확인한다.

### 복습 질문 및 답변

**Q1. `map`의 결과를 변수에 받지 않아도 원본이 바뀌나요?**

<details>
<summary>답</summary>

일반적으로 새 Dataset을 반환하므로 결과를 저장해 다음 단계에 사용해야 합니다.

</details>

**Q2. 캐시가 재현성을 자동 보장하나요?**

<details>
<summary>답</summary>

아닙니다. 데이터 revision, 변환 코드와 라이브러리 버전도 함께 기록해야 합니다.

</details>

**Q3. filter 후 라벨 분포를 다시 봐야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

제거 조건이 특정 라벨에 더 많이 작용해 데이터 편향을 바꿀 수 있기 때문입니다.

</details>

## 한 줄 정리

> Datasets는 split·스키마·변환 이력을 중심으로 대규모 전처리를 재사용 가능한 파이프라인으로 만듭니다.
