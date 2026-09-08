# 학습 안정성 진단과 디버깅

긴 모델 학습은 `loss` 하나만 보고 운영하기 어렵습니다. 손실, gradient norm, 학습률, 입력 데이터를 함께 관측해야 문제를 재현하고 원인을 좁힐 수 있습니다.

**핵심 키워드:** loss divergence, gradient norm, clipping, warmup, observability

## 안정적인 학습이란

안정적이라는 말은 손실이 매 step 감소한다는 뜻이 아닙니다. 일시적인 흔들림이 있어도 수치가 유한하고, 업데이트 크기가 통제되며, 같은 조건에서 비슷한 경향을 재현할 수 있어야 합니다.

```text
입력·라벨 검사 → forward → loss 검사 → backward
→ gradient norm 기록 → clipping → optimizer step → scheduler step
```

문제가 생기면 모델 구조보다 먼저 데이터 샘플, 라벨 범위, 학습률, 정밀도 설정을 확인합니다. `NaN`이나 `Inf`가 처음 나타난 step과 직전 로그를 남기면 원인 범위를 크게 줄일 수 있습니다.

## Gradient clipping과 학습률

전체 gradient의 L2 norm이 임계값 $c$보다 클 때 다음처럼 크기만 줄입니다.

$$
g \leftarrow g \times \frac{c}{\lVert g \rVert_2}
$$

방향은 유지하고 크기만 제한하므로 갑작스러운 폭주를 완화합니다. 다만 clipping은 잘못된 라벨이나 과도한 학습률의 근본 원인을 고치는 도구가 아닙니다.

```python
optimizer.zero_grad(set_to_none=True)
loss = model(**batch).loss
loss.backward()

norm_before = torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
optimizer.step()
scheduler.step()
```

`clip_grad_norm_()`의 반환값은 clipping 이전 norm입니다. 호출 순서는 `backward → clipping → optimizer.step → scheduler.step`으로 두는 것이 핵심입니다.

## 로그를 읽는 기준

| 관측 패턴 | 우선 확인할 것 |
|---|---|
| loss와 norm이 함께 급등 | 학습률, 이상 배치, 라벨 |
| loss가 NaN인데 norm은 정상 | 손실 계산, 입력 범위, mixed precision |
| norm이 계속 임계값에 걸림 | 학습률 과다, 임계값 과소 |
| 학습은 안정적이나 개선 없음 | 데이터 신호, 모델 용량, 학습률 과소 |

한 번에 여러 설정을 바꾸지 말고 작은 데이터와 짧은 step으로 가설 하나씩 검증합니다.

## 직접 해보기

1. clipping 전 norm이 4이고 임계값이 1이면 gradient 크기는 몇 배가 되나요?
2. scheduler를 optimizer보다 먼저 호출할 때 생길 수 있는 문제를 설명하세요.
3. loss가 갑자기 NaN이 된 실험의 최소 디버깅 로그를 설계하세요.

<details>
<summary>정답 보기</summary>

1. 방향을 유지한 채 원래 크기의 1/4로 축소됩니다.
2. 현재 update에 의도한 학습률 대신 다음 step의 학습률이 적용되어 스케줄이 한 칸 어긋날 수 있습니다.
3. step, batch 식별자, loss, learning rate, clipping 전후 norm, 입력·라벨 범위, 정밀도 설정을 함께 기록합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 안정성 vs 성능 | 계산이 통제되는가 vs 검증 지표가 좋은가 |
| clipping vs learning rate 감소 | 한 step의 gradient 제한 vs 전체 update 비율 조정 |
| loss 변동 vs loss 발산 | 정상적인 잡음 vs 회복되지 않는 급증·NaN |

## 연결되는 개념

- 다음: [체크포인트 복구와 메모리 최적화](02-checkpoints-and-memory.md)
- 함께 볼 키워드: `gradient accumulation`, `mixed precision`, `seed`

## 셀프 체크

- [ ] 학습 안정성과 성능을 구분한다.
- [ ] clipping 호출 순서를 설명한다.
- [ ] gradient norm 로그를 해석한다.
- [ ] NaN 발생 시 데이터부터 점검한다.
- [ ] 한 번에 하나의 가설만 검증한다.

### 복습 질문 및 답변

**Q1. loss만 기록하면 왜 부족한가요?**

<details>
<summary>답</summary>

loss는 결과만 보여 주므로 업데이트 크기, 학습률, 이상 배치 같은 원인을 분리하기 어렵습니다.

</details>

**Q2. clipping을 적용하면 발산이 완전히 사라지나요?**

<details>
<summary>답</summary>

아닙니다. 큰 gradient를 제한할 뿐 데이터 오류나 수치 불안정의 원인은 별도로 해결해야 합니다.

</details>

**Q3. 안정화 설정의 효과는 어떻게 비교하나요?**

<details>
<summary>답</summary>

같은 데이터 분할과 seed를 유지하고 loss, norm, 처리량, 검증 지표를 기준선과 나란히 비교합니다.

</details>

## 한 줄 정리

> 학습 디버깅은 감으로 설정을 바꾸는 일이 아니라 관측 가능한 신호로 실패 원인을 좁히는 과정입니다.
