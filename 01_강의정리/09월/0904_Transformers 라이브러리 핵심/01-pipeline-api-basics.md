# Pipeline API로 빠르게 추론하기

`pipeline()`은 전처리, 모델 실행과 후처리를 하나로 묶어 모델을 빠르게 시험하게 합니다. 짧은 코드의 편리함 뒤에서 어떤 과업과 모델이 선택되는지는 직접 확인해야 합니다.

**핵심 키워드:** pipeline, task, preprocessing, inference, postprocessing

## 기본 흐름

```python
from transformers import pipeline

classifier = pipeline(
    task="text-classification",
    model="distilbert-base-uncased-finetuned-sst-2-english",
)

result = classifier("The explanation was easy to follow.")
print(result)
```

Pipeline은 입력을 토큰화하고 tensor를 모델에 전달한 뒤 logits를 라벨과 점수로 바꿉니다. 출력 점수는 모델의 예측 신뢰도이지 문장의 객관적 진실 확률은 아닙니다.

## 과업 선택

| 과업 | 대표 입력 | 대표 출력 |
|---|---|---|
| text classification | 문자열 | 라벨과 점수 |
| token classification | 문자열 | 토큰별 라벨 |
| question answering | 질문과 문맥 | 답변 span과 점수 |
| text generation | prompt 또는 채팅 | 생성 텍스트 |

기본 모델에 의존하지 말고 검토한 모델 ID와 revision을 명시합니다. 과업과 head가 맞지 않으면 로딩 오류나 의미 없는 출력이 생길 수 있습니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| task vs model | 처리 계약 vs 실제 가중치·구조 |
| pipeline vs model forward | 통합 전후처리 vs tensor 수준 직접 호출 |
| score vs 검증 성능 | 개별 출력 신뢰도 vs 데이터셋 전체 평가 결과 |

## 직접 해보기

1. 감성 분류 Pipeline의 숨은 세 단계를 적으세요.
2. 모델을 명시해야 하는 이유를 설명하세요.
3. 여러 입력을 batch로 전달할 때 측정할 값을 정하세요.

<details>
<summary>정답 보기</summary>

1. 토큰화, 모델 forward, logits 후처리입니다.
2. 기본값 변경을 피하고 라이선스·언어·성능을 검토한 모델을 재현하기 위해서입니다.
3. 처리량, 항목당 지연, 메모리와 출력 순서를 확인합니다.

</details>

## 연결되는 개념

- 다음: [Pipeline 텍스트 생성 제어](02-pipeline-text-generation.md)
- 함께 볼 키워드: `task head`, `batch`, `revision`

## 셀프 체크

- [ ] Pipeline의 세 단계를 설명한다.
- [ ] 과업과 모델의 관계를 구분한다.
- [ ] 모델 ID와 revision을 명시한다.
- [ ] 출력 score를 과대 해석하지 않는다.
- [ ] batch 성능을 측정한다.

### 복습 질문 및 답변

**Q1. Pipeline은 모델 내부를 몰라도 항상 안전하게 쓸 수 있나요?**

<details>
<summary>답</summary>

아닙니다. 입력 제한, 라벨 의미, 모델 카드와 출력 해석을 확인해야 합니다.

</details>

**Q2. 다른 modality도 같은 `pipeline()`을 쓰나요?**

<details>
<summary>답</summary>

같은 진입 함수를 쓸 수 있지만 과업에 따라 이미지·오디오 등 전처리기와 입출력 계약이 달라집니다.

</details>

**Q3. Pipeline 결과를 운영 판단에 바로 써도 되나요?**

<details>
<summary>답</summary>

자체 데이터의 품질·편향·지연·실패 처리를 검증한 뒤 사용해야 합니다.

</details>

## 한 줄 정리

> Pipeline은 빠른 시작점이며 모델·과업·후처리의 실제 의미를 확인할 책임은 사용자에게 있습니다.
