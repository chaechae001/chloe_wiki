# 멀티태스크 학습과 앙상블

멀티태스크는 하나의 백본이 여러 목표를 함께 학습하고, 앙상블은 여러 모델의 예측을 결합합니다. 둘 다 성능을 높일 수 있지만 결합 위치와 오류 상관관계가 다릅니다.

**핵심 키워드:** multitask, ensemble, shared backbone, loss weighting, calibration

## 멀티태스크 구조

```python
class MultiTaskHead(nn.Module):
    def __init__(self, hidden_size, classes):
        super().__init__()
        self.category = nn.Linear(hidden_size, classes)
        self.score = nn.Linear(hidden_size, 1)

    def forward(self, pooled):
        return {
            "category_logits": self.category(pooled),
            "score": self.score(pooled).squeeze(-1),
        }
```

전체 손실은 보통 $L = \lambda_1 L_1 + \lambda_2 L_2$처럼 결합합니다. 가중치는 숫자 크기가 큰 손실 하나가 학습을 지배하지 않도록 검증합니다. 태스크별 데이터가 다르면 어떤 batch에 어떤 loss를 계산할지도 정해야 합니다.

## 앙상블 구조

같은 라벨 순서를 가진 모델의 logits나 확률을 평균할 수 있습니다. 확률 평균은 각 모델의 calibration에 영향을 받고, logits 평균도 모델별 점수 척도가 크게 다르면 편향될 수 있습니다. 검증 데이터로 가중치를 정하고 test 데이터로 조정하지 않습니다.

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 멀티태스크 vs 앙상블 | 한 모델의 공유 표현·여러 head vs 여러 모델 출력 결합 |
| hard voting vs soft voting | 최종 label 다수결 vs 확률·점수 평균 |
| loss weight vs ensemble weight | 학습 목표의 비중 vs 추론 시 모델의 비중 |

## 직접 해보기

1. 분류와 회귀를 함께 학습하는 loss를 설계하세요.
2. 앙상블 전 확인할 label 조건을 적으세요.
3. 오류가 거의 같은 두 모델의 앙상블 효과를 예상하세요.

<details>
<summary>정답 보기</summary>

1. CrossEntropy와 회귀 손실을 각각 계산하고 검증된 가중합을 사용합니다.
2. 클래스 수, label 순서와 전처리·입력 의미를 맞춥니다.
3. 오류 다양성이 작아 개선 폭도 제한적일 가능성이 큽니다.

</details>

## 연결되는 개념

- 이전: [사전학습 백본에 커스텀 헤드 추가하기](01-custom-model-heads.md)
- 다음: [Datasets 구조와 핵심 연산](03-datasets-core-operations.md)

## 셀프 체크

- [ ] 두 결합 방식을 구분한다.
- [ ] 태스크별 출력 shape를 설명한다.
- [ ] loss 가중치 영향을 점검한다.
- [ ] 앙상블 label 순서를 맞춘다.
- [ ] test 누수를 피한다.

### 복습 질문 및 답변

**Q1. 멀티태스크 학습의 장점은 무엇인가요?**

<details>
<summary>답</summary>

관련 과업이 표현을 공유해 데이터 효율과 일반화가 좋아질 수 있습니다.

</details>

**Q2. 관련 없는 과업도 함께 학습하면 좋은가요?**

<details>
<summary>답</summary>

아닙니다. gradient 충돌로 한 과업의 성능이 낮아지는 negative transfer가 생길 수 있습니다.

</details>

**Q3. 앙상블이 항상 단일 모델보다 빠른가요?**

<details>
<summary>답</summary>

대개 여러 모델 추론이 필요해 지연과 메모리 비용이 증가합니다.

</details>

## 한 줄 정리

> 멀티태스크는 표현과 학습을 공유하고 앙상블은 예측을 결합하므로 서로 다른 비용과 검증 기준이 필요합니다.
