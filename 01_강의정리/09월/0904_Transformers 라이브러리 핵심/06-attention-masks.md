# Attention mask 이해

Attention mask는 어떤 토큰 위치를 attention 계산에 포함할지 표현합니다. Padding mask와 Causal mask는 목적이 다르며 decoder 모델에서는 두 제약이 함께 적용될 수 있습니다.

**핵심 키워드:** attention mask, padding mask, causal mask, position, batch

## 두 종류의 제약

| mask | 막는 대상 | 목적 |
|---|---|---|
| Padding mask | 채워 넣은 PAD 위치 | 의미 없는 토큰 무시 |
| Causal mask | 현재보다 미래 위치 | 다음 토큰 학습에서 미래 누출 방지 |

```python
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "left"

batch = tokenizer(
    ["Short", "A somewhat longer prompt"],
    padding=True,
    return_tensors="pt",
)

print(batch["input_ids"])
print(batch["attention_mask"])
```

일반적인 tokenizer mask에서 실제 토큰은 1, padding은 0입니다. 생성 모델의 padding 방향과 PAD 토큰은 모델·tokenizer 설정을 따릅니다. PAD를 EOS로 대체할 때도 종료 판정과 학습 loss mask가 의도대로인지 검증합니다.

## 직접 forward

`model(input_ids=..., attention_mask=...)`처럼 mask를 함께 전달합니다. Mask를 빼면 batch 안의 padding을 실제 문맥처럼 처리할 수 있습니다. 직접 cache 루프를 작성할 때는 mask 길이가 과거 cache와 현재 토큰을 모두 덮도록 갱신되어야 합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| padding mask vs causal mask | PAD 제외 vs 미래 토큰 차단 |
| token ID 0 vs mask 0 | vocabulary의 정수 값 vs 해당 위치를 가리는 표시 |
| left padding vs right padding | 앞쪽 채움 vs 뒤쪽 채움 |

## 직접 해보기

1. 두 문장의 mask를 손으로 작성하세요.
2. mask를 누락한 batch 결과를 예상하세요.
3. cache가 있을 때 mask 길이를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 실제 토큰에는 1, 정렬을 위해 채운 위치에는 0을 둡니다.
2. PAD 위치가 문맥에 영향을 주어 길이별 출력이 왜곡될 수 있습니다.
3. 일반적으로 과거 KV 길이와 새 입력 길이를 합친 위치를 덮어야 합니다.

</details>

## 연결되는 개념

- 이전: [생성 전략과 스트리밍](05-generation-and-streaming.md)
- 다음: [KV Cache와 추론 최적화](07-kv-cache-optimization.md)

## 셀프 체크

- [ ] 두 mask의 목적을 구분한다.
- [ ] tokenizer mask를 읽는다.
- [ ] padding 방향을 확인한다.
- [ ] model에 mask를 전달한다.
- [ ] cache와 mask 길이를 연결한다.

### 복습 질문 및 답변

**Q1. attention mask의 0은 항상 토큰 ID 0인가요?**

<details>
<summary>답</summary>

아닙니다. mask 값과 vocabulary의 token ID는 서로 다른 배열과 의미입니다.

</details>

**Q2. Causal mask가 없으면 무엇이 문제인가요?**

<details>
<summary>답</summary>

학습 시 현재 위치가 미래 정답 토큰을 볼 수 있어 다음 토큰 예측 목적이 깨집니다.

</details>

**Q3. 모든 생성 모델이 left padding만 허용하나요?**

<details>
<summary>답</summary>

아닙니다. 모델과 구현의 권장 설정을 확인하고 실제 batch 출력으로 검증해야 합니다.

</details>

## 한 줄 정리

> Attention mask는 실제 문맥과 허용된 시간 방향만 attention에 참여시키는 제어 정보입니다.
