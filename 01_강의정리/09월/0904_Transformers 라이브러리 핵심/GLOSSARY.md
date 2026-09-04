# 용어집

| 용어 | 설명 |
|---|---|
| Pipeline | 전처리·모델 실행·후처리를 묶은 고수준 추론 API |
| Task | 분류, 질의응답, 생성처럼 모델이 수행할 작업 계약 |
| Causal LM | 앞 토큰을 바탕으로 다음 토큰을 예측하는 언어 모델 |
| `input_ids` | 토큰을 vocabulary의 정수 ID로 바꾼 tensor |
| Logit | softmax 적용 전 토큰 후보의 실수 점수 |
| Softmax | 점수를 합이 1인 분포로 정규화하는 함수 |
| Hidden state | 각 토큰 위치의 문맥 표현 벡터 |
| LM head | hidden state를 vocabulary logits로 투영하는 계층 |
| Greedy decoding | 매 단계 최고 점수 토큰을 선택하는 방식 |
| Sampling | 확률 분포에서 다음 토큰을 뽑는 방식 |
| Beam search | 여러 후보 시퀀스를 유지하며 탐색하는 방식 |
| Temperature | sampling 분포의 평탄함을 조절하는 값 |
| Top-k | 높은 점수의 k개 토큰으로 후보를 제한하는 방법 |
| Top-p | 누적 확률이 p에 도달하는 최소 후보 집합을 쓰는 방법 |
| Streamer | 생성되는 텍스트 조각을 순차적으로 소비하게 하는 도구 |
| Attention mask | attention에 참여할 토큰 위치를 나타내는 정보 |
| Causal mask | 미래 토큰 위치를 보지 못하게 하는 mask |
| KV Cache | 이전 토큰의 attention key와 value를 재사용하는 상태 |
| `past_key_values` | 모델 출력·입력에서 KV Cache를 전달하는 인터페이스 |
| dtype | tensor 원소의 수치 자료형과 정밀도 |
| Quantization | 가중치·연산을 저비트 표현으로 줄이는 기법 |
| Offload | 일부 모델·cache를 CPU나 저장장치로 옮기는 방식 |

## 함께 보기

- [AutoModelForCausalLM 직접 추론](03-causal-lm-direct-inference.md)
- [logits와 hidden state 해석](04-logits-hidden-states.md)
- [KV Cache와 추론 최적화](07-kv-cache-optimization.md)
