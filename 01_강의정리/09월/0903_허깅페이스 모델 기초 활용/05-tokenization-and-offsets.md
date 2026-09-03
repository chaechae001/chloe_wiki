# 토큰화와 offset mapping

토크나이저는 문자열을 모델 vocabulary의 ID 배열로 바꿉니다. 분절 알고리즘과 정규화 규칙이 다르면 같은 문장도 다른 입력이 되며, offset은 토큰 결과를 원문 위치로 되돌리는 다리입니다.

**핵심 키워드:** BPE, WordPiece, SentencePiece, Unigram, special token, offset

## 알고리즘 관점

| 방식 | 핵심 아이디어 |
|---|---|
| BPE | 자주 함께 나타나는 단위를 반복 병합 |
| WordPiece | vocabulary와 확률적 기준으로 부분 단어 구성 |
| SentencePiece | 공백을 포함한 원문에서 독립적으로 subword 학습 |
| Unigram | 후보 조각 집합에서 문장 확률이 좋은 구성을 선택 |

이름만으로 실제 구현을 단정하지 말고 해당 토크나이저의 파일과 문서를 확인합니다.

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased", use_fast=True)
text = "Tokenization connects text and tokens."
encoded = tokenizer(text, return_offsets_mapping=True)

for token_id, (start, end) in zip(encoded["input_ids"], encoded["offset_mapping"]):
    print(tokenizer.convert_ids_to_tokens(token_id), repr(text[start:end]))
```

특수 토큰은 원문 범위가 없어 `(0, 0)`처럼 표시될 수 있습니다. offset은 개체명 표시, 질의응답 span과 오류 분석에 활용합니다.

## Fast와 Slow

Fast tokenizer는 Rust 기반 구현을 사용하며 batch 처리와 offset mapping 같은 정렬 정보를 제공하는 경우가 많습니다. 모델별 지원 범위를 확인합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| token vs token ID | 사람이 보는 분절 문자열 vs vocabulary의 정수 인덱스 |
| encode vs decode | 텍스트를 ID로 변환 vs ID를 텍스트로 복원 |
| character index vs token index | 원문 문자 위치 vs 토큰 배열 위치 |

## 직접 해보기

1. 특수 토큰의 offset이 비어 있는 이유를 설명하세요.
2. 같은 단어가 문맥에 따라 다르게 분절될 수 있는지 확인하세요.
3. 원문 단어와 겹치는 토큰을 찾는 조건을 작성하세요.

<details>
<summary>정답 보기</summary>

1. 모델 입력을 위해 추가됐지만 원문에는 대응 문자가 없기 때문입니다.
2. 정규화와 앞 공백·위치 표시에 따라 토큰 문자열이 달라질 수 있습니다.
3. 토큰 범위의 끝이 단어 시작보다 크고 시작이 단어 끝보다 작은지 검사합니다.

</details>

## 연결되는 개념

- 이전: [state_dict와 파라미터 분석](04-state-dict-parameter-analysis.md)
- 다음: [배치, 패딩과 트런케이션](06-padding-truncation-attention-mask.md)

## 셀프 체크

- [ ] 텍스트→토큰→ID 흐름을 설명한다.
- [ ] 주요 subword 방식을 구분한다.
- [ ] 특수 토큰의 역할을 안다.
- [ ] Fast tokenizer의 장점을 설명한다.
- [ ] offset으로 원문 범위를 찾는다.

### 복습 질문 및 답변

**Q1. 토큰 수와 단어 수가 다른 이유는 무엇인가요?**

<details>
<summary>답</summary>

단어가 여러 subword로 나뉘거나 구두점·특수 토큰이 별도 항목이 되기 때문입니다.

</details>

**Q2. `[UNK]`는 무엇을 뜻하나요?**

<details>
<summary>답</summary>

토크나이저 vocabulary와 규칙으로 표현하지 못한 입력을 나타내는 특수 토큰입니다.

</details>

**Q3. offset mapping이 모든 토크나이저에서 같은 방식인가요?**

<details>
<summary>답</summary>

아닙니다. 정규화와 구현 차이가 있으므로 사용 모델의 반환 규칙을 실제 입력으로 확인해야 합니다.

</details>

## 한 줄 정리

> 토크나이저는 원문을 모델 ID로 바꾸고 offset mapping은 결과를 다시 원문 증거와 연결합니다.
