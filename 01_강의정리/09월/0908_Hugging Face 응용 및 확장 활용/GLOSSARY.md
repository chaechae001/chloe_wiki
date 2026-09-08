# 용어집

Hugging Face 모델 최적화와 배포에서 자주 만나는 용어를 빠르게 찾을 수 있도록 정리했습니다.

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| loss divergence | 손실이 통제 범위를 벗어나 급증하거나 NaN이 되는 현상 | [01](01-training-stability-debugging.md) |
| gradient norm | 전체 gradient 크기를 하나의 값으로 요약한 지표 | [01](01-training-stability-debugging.md) |
| gradient clipping | 임계값을 넘는 gradient 크기를 비례 축소하는 기법 | [01](01-training-stability-debugging.md) |
| warmup | 초반 학습률을 점진적으로 높이는 구간 | [01](01-training-stability-debugging.md) |
| checkpoint | 학습을 저장·재개하기 위한 모델과 optimizer 등의 상태 묶음 | [02](02-checkpoints-and-memory.md) |
| RNG state | 난수 생성 흐름의 현재 상태 | [02](02-checkpoints-and-memory.md) |
| mixed precision | 여러 수치 정밀도를 섞어 메모리와 연산을 최적화하는 방식 | [02](02-checkpoints-and-memory.md) |
| gradient checkpointing | activation 저장을 줄이고 backward 때 재계산하는 방식 | [02](02-checkpoints-and-memory.md) |
| PEFT | 일부 파라미터만 학습하는 효율적 미세조정 방법의 범주 | [03](03-peft-and-lora.md) |
| LoRA | 가중치 변화량을 두 저랭크 행렬로 학습하는 PEFT 기법 | [03](03-peft-and-lora.md) |
| rank | LoRA 변화량의 중간 차원으로 용량과 비용을 조절하는 값 | [03](03-peft-and-lora.md) |
| adapter | base 모델에 덧붙이는 작은 학습 모듈 | [03](03-peft-and-lora.md) |
| quantization | 가중치·activation을 낮은 정밀도로 표현하는 최적화 | [04](04-quantization-strategies.md) |
| calibration | 정적 양자화 범위를 정하기 위한 대표 데이터 관측 | [04](04-quantization-strategies.md) |
| NF4 | 정규분포형 가중치에 맞춘 4-bit 데이터 형식 | [04](04-quantization-strategies.md) |
| QLoRA | 양자화 base 모델과 LoRA 학습을 결합한 방식 | [04](04-quantization-strategies.md) |
| knowledge distillation | teacher의 분포 지식을 student에 전달하는 학습 | [05](05-distillation-and-evaluation.md) |
| temperature | logits 분포의 부드러움을 조절하는 값 | [05](05-distillation-and-evaluation.md) |
| model card | 모델의 용도·평가·한계·라이선스를 설명하는 문서 | [06](06-hub-packaging-and-model-cards.md) |
| gated model | 접근 요청과 동의가 필요한 Hub 모델 | [07](07-auth-and-gated-models.md) |
| fine-grained token | 특정 자원과 행동에 범위를 좁힌 토큰 | [07](07-auth-and-gated-models.md) |
| least privilege | 작업에 필요한 최소 권한만 부여하는 원칙 | [07](07-auth-and-gated-models.md) |
