# Transformers 라이브러리 핵심

Pipeline의 간편한 추론에서 시작해 AutoModelForCausalLM의 직접 forward, logits·hidden state, 생성 전략과 스트리밍, attention mask와 KV Cache 최적화까지 텍스트 생성의 내부 흐름을 학습합니다.

## 학습 목표

- Pipeline이 수행하는 전처리·추론·후처리를 설명합니다.
- 생성 길이와 greedy·sampling 설정을 구분합니다.
- Causal LM의 logits shape와 다음 토큰 확률을 해석합니다.
- `forward()`와 `generate()`, 스트리밍의 책임을 구분합니다.
- Attention mask와 KV Cache의 속도·메모리 상충 관계를 평가합니다.

## 추천 학습 순서

1. [Pipeline API로 빠르게 추론하기](01-pipeline-api-basics.md)
2. [Pipeline 텍스트 생성 제어](02-pipeline-text-generation.md)
3. [AutoModelForCausalLM 직접 추론](03-causal-lm-direct-inference.md)
4. [logits와 hidden state 해석](04-logits-hidden-states.md)
5. [생성 전략과 스트리밍](05-generation-and-streaming.md)
6. [Attention mask 이해](06-attention-masks.md)
7. [KV Cache와 추론 최적화](07-kv-cache-optimization.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
prompt·messages → tokenizer → input_ids·attention_mask
→ Causal LM forward → vocabulary logits → decoding strategy
→ 새 토큰 추가 → KV Cache 갱신 → 종료 조건 → decode·stream
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| 고수준 추론 API | 01 Pipeline 기본 |
| 생성 길이와 sampling | 02 Pipeline 생성 |
| 직접 forward와 generate | 03 Causal LM |
| softmax, top-k, hidden state | 04 출력 해석 |
| greedy, beam, streamer | 05 생성·스트리밍 |
| padding과 causal mask | 06 Attention mask |
| past key/value와 메모리 | 07 KV Cache |

## 최종 점검

- [ ] 과업과 검토한 모델·revision을 명시한다.
- [ ] 생성 옵션과 random seed 등 실험 조건을 기록한다.
- [ ] logits·hidden state의 shape을 먼저 확인한다.
- [ ] padding·causal mask의 목적을 구분한다.
- [ ] 품질·지연·처리량·peak 메모리를 함께 측정한다.
