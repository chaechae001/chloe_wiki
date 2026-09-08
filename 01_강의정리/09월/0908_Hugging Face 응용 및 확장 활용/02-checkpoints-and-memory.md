# 체크포인트 복구와 메모리 최적화

오래 걸리는 학습은 중단을 전제로 설계해야 합니다. 체크포인트는 가중치뿐 아니라 학습의 진행 상태를 저장하고, 메모리 최적화는 같은 장비에서 가능한 실험 범위를 넓힙니다.

**핵심 키워드:** checkpoint, optimizer state, RNG state, mixed precision, gradient checkpointing

## 복구 가능한 체크포인트

이어 학습에는 다음 상태가 함께 필요합니다.

- 모델 가중치
- optimizer와 scheduler 상태
- epoch·global step
- 난수 생성기 상태와 sampler 진행 정보
- 학습 설정, 데이터 버전, 코드 버전

```python
state = {
    "model": model.state_dict(),
    "optimizer": optimizer.state_dict(),
    "scheduler": scheduler.state_dict(),
    "step": global_step,
    "torch_rng": torch.get_rng_state(),
}
torch.save(state, checkpoint_path)
```

복구할 때는 같은 구조로 모델·optimizer·scheduler를 먼저 만든 뒤 각각의 state dict를 불러옵니다. Adam 계열 optimizer의 이동 평균을 빼면 같은 가중치에서 시작해도 다음 update가 달라집니다.

## 저장이 아니라 복구를 테스트하기

체크포인트 파일이 생겼다는 사실만으로 충분하지 않습니다. 짧은 학습에서 저장하고, 새 프로세스처럼 객체를 다시 만든 다음, step과 학습률이 이어지는지 확인합니다.

```text
N step 학습 → checkpoint 저장 → 객체 재생성 → 모든 state 복원
→ N+1 step 실행 → 연속 실행 결과와 허용 오차 비교
```

보관 정책에는 최근 N개, 최고 검증 지표, 마지막 정상 checkpoint를 구분하는 편이 안전합니다. 저장 중 실패로 불완전한 파일이 남지 않도록 임시 파일에 쓴 뒤 원자적으로 이름을 바꾸는 전략도 유용합니다.

## 메모리 최적화의 교환관계

| 기법 | 줄이는 것 | 비용·주의점 |
|---|---|---|
| mixed precision | 텐서·activation 메모리 | 하드웨어와 수치 범위 확인 |
| gradient accumulation | step당 배치 메모리 | update 간격 증가, 처리 시간 |
| gradient checkpointing | 저장 activation | backward 재계산으로 느려짐 |
| PEFT | 학습 파라미터·optimizer 상태 | 적용 모듈과 품질 검증 |

메모리 사용량만 줄었다고 성공은 아닙니다. wall-clock 시간, 처리량, 검증 품질, 수치 안정성을 같은 조건에서 측정해야 합니다.

## 직접 해보기

1. 모델 가중치만 저장하면 이어 학습에서 무엇이 달라질 수 있나요?
2. gradient checkpointing이 메모리를 줄이는 대신 시간을 쓰는 이유는 무엇인가요?
3. 체크포인트 보관 정책을 `latest`, `best`, `periodic`으로 나눠 설계하세요.

<details>
<summary>정답 보기</summary>

1. optimizer의 momentum과 scheduler 진행 상태가 초기화되어 다음 update가 달라질 수 있습니다.
2. forward의 일부 activation을 저장하지 않고 backward 때 다시 계산하기 때문입니다.
3. 재개용 최신본, 배포 후보인 최고 지표본, 장애 분석용 주기본을 각각 보관하고 수명 정책을 둡니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 모델 저장 vs 학습 체크포인트 | 추론 자산 vs 이어 학습 전체 상태 |
| gradient accumulation vs checkpointing | 작은 배치를 누적 vs activation 재계산 |
| fp16 vs bf16 | 더 좁은 지수 범위 vs 넓은 지수 범위와 하드웨어 요구 |

## 연결되는 개념

- 이전: [학습 안정성 진단과 디버깅](01-training-stability-debugging.md)
- 다음: [PEFT와 LoRA 어댑터](03-peft-and-lora.md)
- 함께 볼 키워드: `resume`, `atomic write`, `throughput`

## 셀프 체크

- [ ] 복구에 필요한 상태를 열거한다.
- [ ] optimizer state의 필요성을 설명한다.
- [ ] 복구 테스트를 설계한다.
- [ ] 메모리와 속도의 교환관계를 이해한다.
- [ ] best와 latest 체크포인트를 구분한다.

### 복습 질문 및 답변

**Q1. 같은 checkpoint면 결과도 완전히 같나요?**

<details>
<summary>답</summary>

데이터 순서, RNG, 라이브러리·하드웨어의 비결정적 연산이 다르면 결과가 달라질 수 있습니다.

</details>

**Q2. mixed precision은 항상 빠른가요?**

<details>
<summary>답</summary>

아닙니다. 하드웨어 지원, 연산 구성, 데이터 이동 비용에 따라 이득이 달라져 실제 측정이 필요합니다.

</details>

**Q3. 체크포인트를 많이 남기면 안전하기만 한가요?**

<details>
<summary>답</summary>

저장 비용과 식별 혼선이 커지므로 보존 개수, 명명 규칙, 검증 완료 여부를 함께 관리해야 합니다.

</details>

## 한 줄 정리

> 신뢰할 수 있는 학습은 중단 후 같은 상태로 돌아올 수 있고, 최적화 효과를 품질·속도와 함께 측정합니다.
