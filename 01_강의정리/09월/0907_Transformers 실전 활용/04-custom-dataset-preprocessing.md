# 커스텀 데이터셋과 토큰화 전처리

커스텀 데이터셋은 파일을 읽는 것보다 열의 의미, label 타입, split 누수와 토큰화 결과를 명확히 정의하는 일이 중요합니다.

**핵심 키워드:** CSV, JSON, Parquet, Features, batched map, data collator

## 입력 포맷 선택

| 포맷 | 장점 | 적합한 상황 |
|---|---|---|
| CSV | 단순하고 확인하기 쉬움 | 평평한 소규모 표 |
| JSON/JSONL | 중첩 구조 표현 | 대화·복합 레코드 |
| Parquet | 열 기반 압축과 선택 읽기 | 큰 정형 데이터 |

업로드 전 개인정보, 라이선스, train/test 중복과 label 정의를 검사합니다. 파일 확장자만으로 데이터가 안전하거나 품질이 좋다고 판단하지 않습니다.

```python
def tokenize_batch(batch):
    return tokenizer(
        batch["text"],
        truncation=True,
        max_length=128,
    )

tokenized = dataset.map(tokenize_batch, batched=True)
print(tokenized["train"].column_names)
```

동적 padding은 모든 샘플을 고정 최대 길이로 미리 채우지 않고 data collator가 현재 batch 최장 길이에 맞추게 합니다. 저장 공간과 연산 낭비를 줄일 수 있습니다.

## 검증 순서

1. `features`와 실제 샘플을 확인합니다.
2. 결측·중복·label 범위를 검사합니다.
3. split 간 동일 문서·사용자 누수를 검사합니다.
4. 토큰 길이 분포와 잘림 비율을 측정합니다.
5. 전처리 뒤 모델 입력 열과 dtype을 확인합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| batched map vs batch training | 전처리 함수의 묶음 처리 vs optimizer 단계의 학습 batch |
| static vs dynamic padding | 고정 길이로 미리 채움 vs batch 생성 시 길이 맞춤 |
| validation vs test | 튜닝 중 선택 근거 vs 마지막 일반화 평가 |

## 직접 해보기

1. 토큰화 후 필요한 열을 나열하세요.
2. truncation 손실을 측정하는 방법을 설계하세요.
3. 사용자 단위 split이 필요한 사례를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 과업에 따라 `input_ids`, `attention_mask`, `labels` 등이 필요합니다.
2. 원 토큰 길이와 제한 길이를 비교해 잘린 샘플 비율과 제거 토큰 수를 집계합니다.
3. 같은 사용자의 문체가 여러 split에 섞이면 과대평가될 수 있는 개인화 텍스트가 예입니다.

</details>

## 연결되는 개념

- 이전: [Datasets 구조와 핵심 연산](03-datasets-core-operations.md)
- 다음: [Evaluate로 평가 파이프라인 만들기](05-evaluate-metrics-pipeline.md)

## 셀프 체크

- [ ] 포맷을 데이터 구조에 맞게 고른다.
- [ ] features와 label을 검증한다.
- [ ] split 누수를 검사한다.
- [ ] batch 토큰화를 적용한다.
- [ ] truncation 비율을 측정한다.

### 복습 질문 및 답변

**Q1. 토큰화할 때 padding을 생략해도 되나요?**

<details>
<summary>답</summary>

동적 padding을 data collator에서 수행한다면 전처리 단계에서는 생략할 수 있습니다.

</details>

**Q2. 원문 열을 항상 제거해야 하나요?**

<details>
<summary>답</summary>

아닙니다. 디버깅에는 유용하지만 Trainer가 모델에 불필요한 열을 전달하지 않도록 처리 정책을 확인합니다.

</details>

**Q3. Hub 업로드 전에 가장 먼저 볼 것은 무엇인가요?**

<details>
<summary>답</summary>

배포 권리와 개인정보·민감 정보가 포함됐는지 확인하는 것이 우선입니다.

</details>

## 한 줄 정리

> 커스텀 데이터 전처리는 파일 변환이 아니라 스키마·누수·길이·배치 계약을 검증하는 과정입니다.
