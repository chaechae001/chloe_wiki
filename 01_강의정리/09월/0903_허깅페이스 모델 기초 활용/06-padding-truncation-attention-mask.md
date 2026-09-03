# 배치, 패딩과 트런케이션

길이가 다른 문장을 하나의 텐서로 묶으려면 짧은 입력은 채우고 긴 입력은 제한해야 합니다. attention mask는 채운 위치를 모델 계산에서 구분하도록 알려줍니다.

**핵심 키워드:** batch, padding, truncation, max_length, attention_mask

## 배치 정규화

```python
texts = [
    "Short input.",
    "This input contains several more tokens than the first one.",
]

batch = tokenizer(
    texts,
    padding=True,
    truncation=True,
    max_length=12,
    return_tensors="pt",
)

print(batch["input_ids"].shape)
print(batch["attention_mask"])
```

두 문장이 같은 길이의 직사각형 텐서가 됩니다. `attention_mask`의 1은 실제 입력, 0은 일반적으로 padding 위치를 뜻합니다. 모델 구현에 따라 다른 mask도 있으므로 문서를 확인합니다.

## 전략 선택

- `padding=True`: 현재 batch에서 가장 긴 문장에 맞춥니다.
- `padding="max_length"`: 지정하거나 모델이 허용한 최대 길이에 맞춥니다.
- `truncation=True`: 너무 긴 입력을 최대 길이에 맞게 자릅니다.
- 문장 쌍은 `only_first`, `only_second` 같은 전략이 필요할 수 있습니다.

트런케이션은 정보 손실입니다. 문서 분류에서는 핵심 내용이 뒤에 있다면 단순 자르기 대신 chunk와 겹침, 검색 기반 선택을 검토합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| padding vs truncation | 짧은 입력을 채움 vs 긴 입력을 자름 |
| longest vs max_length | 현재 batch 기준 vs 고정 최대 길이 기준 |
| token length vs character length | 토큰 배열 길이 vs 원문 문자 수 |

## 직접 해보기

1. 세 문장의 batch shape를 예측하세요.
2. padding 토큰을 계산에 포함하면 생길 문제를 설명하세요.
3. 긴 문서의 정보 손실 완화 전략을 세우세요.

<details>
<summary>정답 보기</summary>

1. 행은 문장 수, 열은 padding·truncation 후 토큰 길이입니다.
2. 의미 없는 위치가 표현과 집계에 영향을 주어 결과가 왜곡될 수 있습니다.
3. 겹치는 chunk로 나누고 결과를 집계하거나 관련 구간을 먼저 검색합니다.

</details>

## 연결되는 개념

- 이전: [토큰화와 offset mapping](05-tokenization-and-offsets.md)
- 다음: [채팅 템플릿과 대화 입력](07-chat-templates.md)

## 셀프 체크

- [ ] batch를 직사각형 텐서로 만든다.
- [ ] padding 전략을 구분한다.
- [ ] truncation의 정보 손실을 설명한다.
- [ ] attention mask를 해석한다.
- [ ] 긴 문서 전략을 제안한다.

### 복습 질문 및 답변

**Q1. 단일 문장에 `padding=True`를 쓰면 항상 길어지나요?**

<details>
<summary>답</summary>

현재 batch의 최장 길이에 맞추므로 문장 하나뿐이면 추가 padding이 없을 수 있습니다.

</details>

**Q2. `max_length`만 지정하면 자동으로 잘리나요?**

<details>
<summary>답</summary>

일반적으로 truncation 또는 padding 전략을 함께 활성화해야 의도한 제한이 적용됩니다.

</details>

**Q3. 생성 모델에서 padding 방향이 중요한 이유는 무엇인가요?**

<details>
<summary>답</summary>

다음 토큰을 이어 쓰는 위치와 mask 처리에 영향을 주므로 모델·파이프라인이 기대하는 방향을 사용해야 합니다.

</details>

## 한 줄 정리

> 패딩과 트런케이션은 배치 shape를 맞추고 attention mask는 실제 토큰과 채운 위치를 구분합니다.
