# state_dict와 파라미터 분석

`state_dict()`는 파라미터 이름과 텐서를 대응시킨 사전입니다. 이름과 shape를 읽으면 embedding, attention과 feed-forward가 모델 크기에서 차지하는 비중을 추적할 수 있습니다.

**핵심 키워드:** state_dict, parameter, shape, numel, requires_grad

## 기본 집계

```python
from collections import defaultdict

total = sum(p.numel() for p in model.parameters())
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)

by_root = defaultdict(int)
for name, tensor in model.state_dict().items():
    by_root[name.split(".")[0]] += tensor.numel()

print("total:", total)
print("trainable:", trainable)
print(dict(by_root))
```

`numel()`은 텐서 원소 수입니다. 예를 들어 shape `(768, 768)`의 weight는 `589,824`개 값을 가집니다. 전체 파라미터와 학습 가능 파라미터가 다르면 일부 계층이 동결되었거나 adapter만 학습하는 구성일 수 있습니다.

## attention 파라미터 이해

단순화한 self-attention에는 query, key, value와 output projection이 있어 hidden size가 $d$일 때 weight는 대략 $4d^2$ 규모입니다. bias까지 포함하면 구현에 따라 값이 추가됩니다. 실제 이름은 아키텍처마다 다르므로 문자열 패턴을 고정하기 전에 `named_parameters()`를 관찰합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| parameter vs buffer | 최적화 대상이 될 수 있는 값 vs 이동·저장되지만 보통 학습하지 않는 상태 |
| `state_dict()` vs `parameters()` | 이름 있는 저장 상태 전체 vs Parameter 순회 |
| total vs trainable | 모델의 모든 파라미터 vs gradient로 갱신되는 파라미터 |

## 직접 해보기

1. `(1000, 256)` embedding의 파라미터 수를 계산하세요.
2. 모듈별 비율을 계산하는 코드를 설계하세요.
3. trainable 비율이 1%인 경우를 해석하세요.

<details>
<summary>정답 보기</summary>

1. 256,000개입니다.
2. 이름의 계층 접두사별 `numel()` 합계를 전체 값으로 나눕니다.
3. 대부분이 동결되고 작은 head나 adapter만 학습하도록 구성됐을 가능성을 확인합니다.

</details>

## 연결되는 개념

- 이전: [Auto 클래스와 모델 로딩](03-auto-classes-and-loading.md)
- 다음: [토큰화와 offset mapping](05-tokenization-and-offsets.md)

## 셀프 체크

- [ ] shape에서 `numel`을 계산한다.
- [ ] 모듈별 파라미터를 집계한다.
- [ ] total과 trainable을 구분한다.
- [ ] 이름이 모델마다 다름을 고려한다.
- [ ] 결과를 메모리·학습 전략과 연결한다.

### 복습 질문 및 답변

**Q1. `state_dict`에는 gradient도 항상 저장되나요?**

<details>
<summary>답</summary>

일반적인 모델 state dict는 파라미터와 영속 buffer를 담고 현재 gradient 자체를 기본 저장 대상으로 삼지 않습니다.

</details>

**Q2. 파라미터 수가 많으면 항상 더 좋은 모델인가요?**

<details>
<summary>답</summary>

아닙니다. 데이터, 학습 방법, 과업 적합성, 지연과 비용을 함께 평가해야 합니다.

</details>

**Q3. 접두사만으로 모듈을 집계할 때 주의점은 무엇인가요?**

<details>
<summary>답</summary>

래퍼나 모델 구조에 따라 첫 이름 조각이 실제 의미 있는 계층 구분이 아닐 수 있어 모듈 트리를 함께 확인해야 합니다.

</details>

## 한 줄 정리

> 파라미터 분석은 이름·shape·학습 여부를 집계해 모델 구조와 자원 요구를 수치로 읽는 작업입니다.
