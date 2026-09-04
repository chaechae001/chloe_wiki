# KV Cache와 추론 최적화

Autoregressive 생성은 토큰을 하나씩 추가합니다. KV Cache는 이전 토큰의 attention key와 value를 저장해 매 단계 같은 과거 표현을 다시 계산하는 비용을 줄입니다.

**핵심 키워드:** KV cache, past_key_values, DynamicCache, dtype, device_map

## cache 효과와 비용

```text
cache 없음: 새 토큰마다 전체 과거 구간의 K/V 재계산
cache 사용: 과거 K/V 재사용 + 새 토큰의 K/V만 추가
```

속도는 좋아질 수 있지만 layer, batch, head 수, context 길이에 따라 cache 메모리가 커집니다. 긴 context나 동시 요청에서는 가중치만큼 중요한 메모리 병목이 됩니다.

```python
generated = model.generate(
    **inputs,
    max_new_tokens=40,
    use_cache=True,
)
```

대부분 `generate()`가 mask와 cache를 관리합니다. 직접 루프에서는 `past_key_values`를 다음 forward에 전달하고 입력을 새 토큰으로 줄이며 position과 mask를 정확히 갱신해야 합니다.

## 로딩·생성 최적화 지도

| 옵션 | 기대 효과 | 상충 관계 |
|---|---|---|
| 낮은 `dtype` | 가중치 메모리와 연산량 감소 | 하드웨어 지원·수치 품질 |
| `device_map` | 여러 장치 배치 또는 offload | 전송 비용·지원 연산 |
| `low_cpu_mem_usage` | 로딩 중 CPU peak 완화 | 버전·모델 호환 확인 |
| 양자화 | 더 작은 메모리 | 품질·커널·속도 차이 |
| cache 전략 | 반복 계산 감소 | KV 메모리 증가 |

최적화는 한 번에 하나씩 적용하고 정확도, 첫 토큰 지연, 토큰/초, peak CPU·GPU 메모리를 함께 측정합니다.

## 헷갈리기 쉬운 포인트

| 비교 | 핵심 차이 |
|---|---|
| KV Cache vs 모델 cache | 생성 중 attention 상태 vs 다운로드 파일 cache |
| 속도 최적화 vs 메모리 최적화 | 계산 감소 중심 vs 저장 공간 감소 중심 |
| dtype 축소 vs 양자화 | 일반 부동소수점 정밀도 선택 vs 별도 저비트 표현·스케일 |

## 직접 해보기

1. cache on/off 벤치마크를 설계하세요.
2. OOM 발생 시 조정 순서를 세우세요.
3. 직접 생성 루프의 상태를 나열하세요.

<details>
<summary>정답 보기</summary>

1. 같은 모델·prompt·출력 길이에서 시간과 peak 메모리를 반복 측정합니다.
2. batch·context·출력 길이를 줄이고 dtype·양자화·offload와 cache 전략을 검토합니다.
3. 생성 토큰, attention mask, position, cache와 종료 상태를 관리합니다.

</details>

## 연결되는 개념

- 이전: [Attention mask 이해](06-attention-masks.md)
- 처음으로: [Pipeline API로 빠르게 추론하기](01-pipeline-api-basics.md)

## 셀프 체크

- [ ] KV Cache의 목적을 설명한다.
- [ ] cache의 메모리 비용을 안다.
- [ ] 직접 루프의 상태를 나열한다.
- [ ] 로딩 최적화의 상충 관계를 비교한다.
- [ ] 속도·메모리·품질을 함께 측정한다.

### 복습 질문 및 답변

**Q1. KV Cache는 모델 가중치를 저장하나요?**

<details>
<summary>답</summary>

아닙니다. 현재 생성 요청의 과거 attention key와 value 상태를 저장합니다.

</details>

**Q2. `use_cache=True`면 항상 더 좋은가요?**

<details>
<summary>답</summary>

일반 생성 속도에는 유리하지만 긴 context·큰 batch에서는 메모리 병목이 커져 환경별 측정이 필요합니다.

</details>

**Q3. 최적화 전 기준선이 필요한 이유는 무엇인가요?**

<details>
<summary>답</summary>

변경이 속도·메모리·품질 중 무엇을 얼마나 바꿨는지 분리해 판단하기 위해서입니다.

</details>

## 한 줄 정리

> KV Cache는 과거 attention 계산을 재사용하지만 속도 이득을 위해 메모리를 사용하므로 실제 workload로 선택해야 합니다.
