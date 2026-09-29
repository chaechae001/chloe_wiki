# 응답 캐시와 미니 프로젝트

> 캐시는 동일한 제품 입력에 대한 완성 응답을 재사용하되, key·TTL·사용자 경계·무효화 정책을 함께 설계해야 합니다.

`response cache` · `cache key` · `TTL` · `in-memory` · `mini project`

## 핵심요약

- 앱 응답 캐시는 완성 답변을 저장하고, OpenAI prompt caching은 prefix 처리 작업을 재사용합니다.
- 응답 cache key에는 instruction, 사용자 입력, 모델, 생성 옵션, prompt version이 필요합니다.
- In-memory dict는 학습과 단일 process에는 적합하지만 여러 worker가 공유하지 못합니다.
- TTL과 최대 크기, 개인정보 경계를 정의합니다.
- 미니 프로젝트는 설정→Schema→Client→Service→Router→Main 순서로 조립하고 contract를 테스트합니다.

## 1. 결정론적인 Cache Key

```python
import hashlib
import json

def make_key(payload: dict) -> str:
    canonical = json.dumps(
        payload,
        ensure_ascii=False,
        sort_keys=True,
        separators=(",", ":"),
    )
    return hashlib.sha256(canonical.encode("utf-8")).hexdigest()
```

입력 순서와 serialization이 안정적이어야 같은 요청이 같은 key를 만듭니다. 사용자별 개인화 답변을 공유 cache에 넣지 않도록 tenant/user scope도 고려합니다.

<details>
<summary>답</summary>

System instruction이나 prompt version이 key에서 빠지면 정책을 바꾼 뒤에도 이전 답변이 잘못 재사용될 수 있습니다.

</details>

## 2. 두 종류의 캐시 구분

| 구분 | 저장·재사용 대상 | 앱에서 보이는 동작 |
|---|---|---|
| 응답 cache | 완성된 답변 | Provider 호출 없이 바로 반환 |
| Prompt caching | 안정적인 입력 prefix 처리 | 새 응답을 생성하지만 입력 처리 절감 |

OpenAI의 prompt caching은 지원 모델에서 공통 prefix 재사용을 통해 지연과 입력 비용을 줄일 수 있습니다. 앱 응답 캐시의 정확성·무효화 책임을 대신하지는 않습니다.

## 3. Mini Project 조립

```text
1. core/config, logging
2. schemas/request-response
3. clients/provider adapter
4. prompts/versioned policy
5. services/cache + orchestration
6. routers/HTTP contract
7. main/app assembly
8. tests/success-error-cache-cancel
```

처음부터 실제 LLM을 호출하지 말고 Fake Client로 정상·빈 응답·timeout을 검증한 뒤 제한된 통합 test를 추가합니다.

## 직접 해보기

1. Prompt version을 포함한 cache key를 만드세요.
2. TTL이 지난 항목을 miss로 처리하세요.
3. Fake Client 호출 횟수로 cache hit를 검증하세요.

<details>
<summary>정답 보기</summary>

Canonical JSON을 hash하고 저장 시각과 TTL을 함께 보관합니다. 같은 요청을 두 번 보냈을 때 Fake Client가 한 번만 호출되는지 assertion합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Response cache vs prompt caching | 완성 답변 재사용과 입력 prefix 계산 재사용입니다. |
| Cache hit vs 같은 의미 | Key가 같은 것과 의미가 유사한 것은 다릅니다. |
| In-memory vs shared cache | Process 내부 저장과 여러 instance가 공유하는 저장소입니다. |

## 연결되는 개념

- 이전: [Logging과 관측성](05-logging-and-observability.md)
- 함께 보기: [전체 개요](OVERVIEW.md)

## 셀프 체크

- [ ] 두 캐시 개념을 구분한다.
- [ ] Cache key 필드를 정의한다.
- [ ] TTL과 최대 크기를 둔다.
- [ ] 사용자별 data 경계를 지킨다.
- [ ] Fake Client로 hit·miss를 테스트한다.

### 복습 질문 및 답변

**Q1. Temperature가 다르면 같은 cache key를 써도 되나요?**

<details>
<summary>답</summary>

생성 결과에 영향을 주므로 일반적으로 key에 포함해야 합니다.

</details>

**Q2. In-memory cache가 운영에서 왜 부족한가요?**

<details>
<summary>답</summary>

Process마다 내용이 다르고 재시작 시 사라지며 크기와 만료 관리가 제한적입니다.

</details>

**Q3. Cache된 답변도 logging해야 하나요?**

<details>
<summary>답</summary>

원문 대신 hit 여부, key prefix, latency, prompt version 등을 안전하게 기록해 효과와 오류를 추적합니다.

</details>

## 한 줄 정리

> 캐시는 hash 함수 하나가 아니라 입력 동등성, 만료, 격리와 관측성을 포함한 제품 정책입니다.
