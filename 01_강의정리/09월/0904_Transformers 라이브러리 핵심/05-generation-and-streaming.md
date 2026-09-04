# 생성 전략과 스트리밍

`generate()`는 logits에서 다음 토큰을 선택하고 다시 입력에 붙이는 과정을 종료 조건까지 반복합니다. 스트리밍은 최종 결과를 바꾸는 전략이 아니라 생성된 토큰을 사용자에게 일찍 전달하는 방식입니다.

**핵심 키워드:** greedy, sampling, beam search, stopping, streamer

## 전략 비교

| 전략 | 선택 방식 | 적합한 상황 |
|---|---|---|
| Greedy | 매 단계 최고 점수 | 짧고 결정적인 출력 |
| Sampling | 확률 분포에서 선택 | 다양성이 필요한 생성 |
| Beam search | 여러 후보 경로 유지 | 제한된 탐색이 유용한 과업 |

```python
generated = model.generate(
    **inputs,
    max_new_tokens=32,
    do_sample=True,
    temperature=0.8,
    top_p=0.9,
    repetition_penalty=1.05,
)
```

설정을 늘리기 전에 기준선을 만들고 한 번에 한 요소를 바꿉니다. EOS, 최대 토큰과 사용자 정의 stopping 조건으로 무한·과도한 생성을 막습니다.

## 스트리밍 구조

```python
from threading import Thread
from transformers import TextIteratorStreamer

streamer = TextIteratorStreamer(tokenizer, skip_prompt=True)
kwargs = {**inputs, "streamer": streamer, "max_new_tokens": 24}
worker = Thread(target=model.generate, kwargs=kwargs)
worker.start()

for piece in streamer:
    print(piece, end="", flush=True)
worker.join()
```

생성은 작업 thread에서 돌고 소비자는 iterator로 조각을 받습니다. 운영에서는 취소, timeout, 예외 전달과 thread 수 제한이 필요합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| 생성 속도 vs 첫 토큰 지연 | 전체 처리량 vs 사용자가 처음 결과를 보는 시간 |
| streaming vs sampling | 전달 방식 vs 토큰 선택 방식 |
| EOS vs 길이 제한 | 의미 있는 종료 표식 vs 강제 안전 상한 |

## 직접 해보기

1. greedy 기준 실험을 설계하세요.
2. 스트리밍 취소 시 정리할 자원을 적으세요.
3. 반복 출력 완화 방법을 비교하세요.

<details>
<summary>정답 보기</summary>

1. 모델·prompt·revision을 고정하고 sampling 없이 길이·지연을 기록합니다.
2. 생성 작업, queue·streamer와 GPU 메모리 참조를 정리합니다.
3. prompt 개선, 반복 패널티, n-gram 제한과 종료 조건을 품질 손실과 함께 평가합니다.

</details>

## 연결되는 개념

- 이전: [logits와 hidden state 해석](04-logits-hidden-states.md)
- 다음: [Attention mask 이해](06-attention-masks.md)

## 셀프 체크

- [ ] 생성 루프를 설명한다.
- [ ] 세 디코딩 전략을 구분한다.
- [ ] 종료 조건을 설정한다.
- [ ] 스트리밍과 sampling을 구분한다.
- [ ] 취소·예외 흐름을 설계한다.

### 복습 질문 및 답변

**Q1. streaming이면 모델 계산도 빨라지나요?**

<details>
<summary>답</summary>

반드시 그렇지 않습니다. 토큰을 즉시 보여 체감 지연을 줄이지만 총 계산량은 비슷할 수 있습니다.

</details>

**Q2. beam 수를 늘리면 항상 품질이 좋아지나요?**

<details>
<summary>답</summary>

아닙니다. 계산과 메모리가 늘고 자유 생성에서는 반복적·평균적인 결과가 될 수 있습니다.

</details>

**Q3. `max_new_tokens`가 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>

EOS가 나오지 않거나 너무 늦게 나오는 경우에도 생성 비용과 응답 길이를 제한합니다.

</details>

## 한 줄 정리

> 생성 전략은 토큰 선택을, 스트리밍은 토큰 전달 시점을 결정합니다.
