# 지식 증류와 경량화 평가

지식 증류는 큰 teacher의 출력 분포를 작은 student가 배우게 합니다. 작은 모델을 만드는 것보다 중요한 일은 teacher의 판단 정보를 얼마나 유지했는지 평가하는 것입니다.

**핵심 키워드:** knowledge distillation, teacher, student, temperature, KL divergence

## Hard label과 Soft target

정답 라벨은 한 클래스만 알려 주지만 teacher의 확률 분포에는 클래스 간 유사성이 남습니다. temperature $T$로 logits를 나누면 분포가 부드러워져 낮은 확률 클래스의 관계도 드러납니다.

$$
L = \alpha T^2 D_{KL}\left(p_t^T \parallel p_s^T\right)
+ (1-\alpha)L_{CE}(y, p_s)
$$

$D_{KL}$은 teacher와 student 분포 차이, $L_{CE}$는 실제 라벨에 대한 손실입니다. $T^2$는 temperature로 줄어든 gradient 크기를 보정합니다.

```python
import torch.nn.functional as F

def distill_loss(student_logits, teacher_logits, labels, temperature=2.0, alpha=0.6):
    hard = F.cross_entropy(student_logits, labels)
    student_log_p = F.log_softmax(student_logits / temperature, dim=-1)
    teacher_p = F.softmax(teacher_logits / temperature, dim=-1)
    soft = F.kl_div(student_log_p, teacher_p, reduction="batchmean") * temperature**2
    return alpha * soft + (1 - alpha) * hard
```

teacher는 평가 모드와 `no_grad()`로 실행해 불필요한 gradient를 만들지 않습니다. 두 모델의 label 순서와 토큰화 조건도 같아야 합니다.

## 무엇을 비교해야 하나

| 축 | 대표 측정값 |
|---|---|
| 품질 | accuracy, F1, calibration, 과업별 오류 |
| 크기 | 파라미터 수, 디스크 크기, 런타임 메모리 |
| 속도 | p50·p95 latency, throughput |
| 일치성 | teacher-student KL, 예측 일치율 |

증류 뒤 양자화를 추가할 수도 있지만 각 단계를 따로 평가해야 손실의 원인을 알 수 있습니다. `student 기준선 → 증류 student → 증류+양자화` 순서로 측정하면 효과를 분리하기 쉽습니다.

## Temperature와 alpha

temperature가 너무 낮으면 hard label과 비슷해지고, 너무 높으면 분포가 지나치게 평평해집니다. alpha가 높으면 teacher 모방을 더 중시합니다. 두 값은 검증 세트에서 탐색하며 teacher의 오류를 student가 그대로 배우지 않는지도 확인합니다.

## 직접 해보기

1. temperature가 커질 때 softmax 분포는 어떻게 변하나요?
2. teacher가 잘못된 편향을 가진 경우 증류에는 어떤 위험이 있나요?
3. 세 단계 경량화 실험의 비교표를 설계하세요.

<details>
<summary>정답 보기</summary>

1. 클래스 확률 차이가 줄어 더 부드러운 분포가 됩니다.
2. student가 정답 라벨뿐 아니라 teacher의 편향과 오류 패턴도 모방할 수 있습니다.
3. 원본 student, 증류 student, 증류+양자화 모델에 대해 품질·크기·latency·메모리를 같은 조건으로 기록합니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 증류 vs 양자화 | 지식을 작은 모델에 전달 vs 숫자 표현 정밀도를 축소 |
| teacher logits vs labels | 클래스별 상대 점수 vs 정답 인덱스 |
| 압축률 vs 품질 유지율 | 자원 절감 크기 vs 예측 성능 보존 정도 |

## 연결되는 개념

- 이전: [양자화 전략과 실행 환경](04-quantization-strategies.md)
- 다음: [Hub 배포 패키지와 모델 카드](06-hub-packaging-and-model-cards.md)
- 함께 볼 키워드: `soft target`, `calibration`, `benchmark`

## 셀프 체크

- [ ] hard loss와 soft loss를 구분한다.
- [ ] temperature의 역할을 설명한다.
- [ ] teacher를 gradient 없이 실행한다.
- [ ] 경량화 단계를 분리 평가한다.
- [ ] 비용과 품질 지표를 함께 본다.

### 복습 질문 및 답변

**Q1. teacher는 반드시 더 큰 모델이어야 하나요?**

<details>
<summary>답</summary>

보통 더 강한 모델을 쓰지만 핵심은 student에게 유용한 분포와 표현 신호를 제공하는 것입니다.

</details>

**Q2. 증류 손실만 사용하면 안 되나요?**

<details>
<summary>답</summary>

가능하지만 teacher 오류에 종속될 수 있어 실제 라벨 손실을 함께 쓰는 구성이 일반적인 기준선입니다.

</details>

**Q3. 경량화 모델의 최종 선택 기준은 무엇인가요?**

<details>
<summary>답</summary>

서비스의 품질 하한을 지키면서 메모리·latency·비용 목표를 만족하는 Pareto 지점을 선택합니다.

</details>

## 한 줄 정리

> 증류는 teacher의 분포 지식을 student에 전달하며, 성공 여부는 품질과 운영 비용을 함께 비교해 판단합니다.
