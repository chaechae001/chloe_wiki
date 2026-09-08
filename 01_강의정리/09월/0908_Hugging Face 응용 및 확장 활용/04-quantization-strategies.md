# 양자화 전략과 실행 환경

양자화는 가중치와 activation을 더 낮은 정밀도로 표현해 메모리와 연산 비용을 줄입니다. 핵심은 비트 수가 아니라 목표 하드웨어에서 품질과 지연 시간을 함께 검증하는 것입니다.

**핵심 키워드:** quantization, int8, int4, bitsandbytes, torchao

## 무엇을 줄이는가

FP32 가중치 하나는 보통 32비트를 사용하지만 INT8은 8비트를 사용합니다. 단순 계산상 가중치 저장 공간은 약 1/4이 될 수 있지만 메타데이터와 양자화하지 않는 층 때문에 실제 파일 비율은 달라집니다.

```text
기준 모델 측정 → 양자화 방식 선택 → 변환·로딩
→ 동일 입력 검증 → 품질·메모리·latency 벤치마크 → 배포 판단
```

| 방식 | 준비 | 적합한 상황 |
|---|---|---|
| 동적 양자화 | 실행 중 activation 범위 처리 | 빠른 CPU 기준선 |
| 정적 양자화 | 대표 데이터로 calibration | 고정된 배포 환경 |
| QAT | 학습 중 양자화 오차 반영 | 정확도 보존이 중요한 경우 |
| 4/8-bit 로딩 | 지원 라이브러리·하드웨어 필요 | 큰 Transformer 메모리 절감 |

## Transformers에서 저비트 로딩

```python
import torch
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

quant_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
)

model = AutoModelForCausalLM.from_pretrained(
    model_id,
    quantization_config=quant_config,
    device_map="auto",
)
```

4-bit base 모델의 모든 파라미터를 그대로 학습하는 것이 아니라 추가 파라미터를 학습하는 QLoRA 계열 흐름과 연결합니다. 설치 버전, GPU 지원, compute dtype을 실행 환경에서 확인해야 합니다.

## PyTorch 양자화 API의 변화

이전 `torch.quantization.quantize_dynamic()` 예제는 환경에 따라 deprecation 경고가 날 수 있습니다. 새 프로젝트는 `torchao.quantization.quantize_()`와 대상 backend의 공식 가이드를 우선 확인합니다. 기존 코드가 작동하더라도 배포 수명과 지원 버전을 고려해 이전 계획을 세웁니다.

## 결과 평가

최소한 다음을 같은 입력과 장비에서 비교합니다.

- validation 지표와 예측 일치율
- 모델 파일 크기와 peak memory
- warm-up 이후 p50·p95 latency
- 처리량과 첫 요청 지연
- 지원하지 않는 연산의 fallback 여부

## 직접 해보기

1. 400MB FP32 가중치를 INT8로 표현할 때 이론적 크기는 얼마인가요?
2. 파일 크기가 줄었는데 latency가 느려질 수 있는 이유를 두 가지 쓰세요.
3. 양자화 배포 승인 기준을 설계하세요.

<details>
<summary>정답 보기</summary>

1. 순수 가중치만 비교하면 약 100MB입니다.
2. 하드웨어 커널 미지원으로 fallback이 생기거나 변환·복원 오버헤드가 연산 절감을 넘어설 수 있습니다.
3. 품질 하락 한도, p95 latency, peak memory, 처리량, 대상 장비 호환성의 합격선을 함께 둡니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 양자화 vs mixed precision | 주로 배포·저비트 표현 최적화 vs 학습 연산 정밀도 혼합 |
| 8-bit vs 4-bit | 상대적으로 안정적 vs 더 큰 절감과 높은 검증 부담 |
| 저장 크기 vs 런타임 메모리 | 디스크 자산 크기 vs 실행 중 가중치·activation·캐시 합계 |

## 연결되는 개념

- 이전: [PEFT와 LoRA 어댑터](03-peft-and-lora.md)
- 다음: [지식 증류와 경량화 평가](05-distillation-and-evaluation.md)
- 함께 볼 키워드: `NF4`, `calibration`, `backend`

## 셀프 체크

- [ ] 양자화가 줄이는 대상을 설명한다.
- [ ] 동적·정적·QAT를 구분한다.
- [ ] 저비트 로딩의 환경 조건을 확인한다.
- [ ] 구식 API의 지원 상태를 점검한다.
- [ ] 품질과 latency를 함께 평가한다.

### 복습 질문 및 답변

**Q1. 비트 수를 낮추면 모델이 항상 빨라지나요?**

<details>
<summary>답</summary>

아닙니다. 대상 하드웨어에 최적화된 커널이 없으면 메모리는 줄어도 속도 이득이 없거나 더 느릴 수 있습니다.

</details>

**Q2. logits 차이가 작으면 검증이 끝난 것인가요?**

<details>
<summary>답</summary>

작은 차이도 결정 경계 근처의 클래스를 바꿀 수 있으므로 전체 검증 지표와 실제 트래픽을 확인해야 합니다.

</details>

**Q3. QLoRA에서 무엇을 학습하나요?**

<details>
<summary>답</summary>

저비트로 로드한 base 모델은 고정하고 LoRA 같은 추가 adapter 파라미터를 학습합니다.

</details>

## 한 줄 정리

> 양자화의 성공 기준은 작은 파일이 아니라 목표 환경에서 유지되는 품질과 실제로 줄어든 비용입니다.
