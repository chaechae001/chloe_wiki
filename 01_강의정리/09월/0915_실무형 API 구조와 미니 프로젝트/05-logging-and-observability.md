# Logging과 관측성

> 로그는 요청 하나의 흐름, 비용, 지연, 실패 원인을 사후에 재구성할 수 있도록 구조화해야 합니다.

`logger` · `request ID` · `latency` · `TTFT` · `structured log`

## 핵심요약

- `print` 대신 module logger와 중앙 설정을 사용합니다.
- 요청 ID, 모델 별칭, 결과 category, latency를 구조화해 기록합니다.
- Streaming에서는 TTFT와 전체 완료 시간을 따로 측정합니다.
- API key, Authorization header, 원문 대화는 기본 로그에서 제외합니다.
- Client·Service·Router가 같은 request ID를 공유해야 추적이 이어집니다.

## 1. 중앙 설정

```python
import logging

def setup_logging(level: int = logging.INFO) -> None:
    logging.basicConfig(
        level=level,
        format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    )
```

각 module에서는 `logging.getLogger(__name__)`만 호출합니다. 운영 규모가 커지면 JSON formatter와 trace ID 연동을 사용합니다.

<details>
<summary>답</summary>

공통 설정을 한곳에 두면 level과 handler를 코드 수정 없이 환경별로 바꾸기 쉽습니다.

</details>

## 2. 기록할 field

| Field | 목적 |
|---|---|
| `request_id` | 여러 레이어의 log 연결 |
| `model_alias` | 모델별 품질·비용 비교 |
| `prompt_version` | 결과 재현 |
| `ttft_ms` | 첫 조각 체감 속도 |
| `elapsed_ms` | 전체 처리 시간 |
| `result` | success, empty, timeout, cancelled, error |
| `cached` | 앱 응답 캐시 hit 여부 |

## 3. 민감정보와 오류

예외 원문에는 URL, key, 사용자 입력이 섞일 수 있습니다. 외부 응답에는 안정적인 오류 code를 제공하고 내부 log에도 필요한 category와 제한된 context만 기록합니다.

## 직접 해보기

1. request ID를 Router에서 생성해 Service로 넘기세요.
2. TTFT와 전체 시간을 따로 측정하세요.
3. 로그 금지 항목 네 가지를 적으세요.

<details>
<summary>정답 보기</summary>

Router가 UUID를 만들고 context 또는 매개변수로 전달합니다. API key, 인증 header, 사용자 원문, provider 오류 전문은 기본 로그에서 제외합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Logging vs monitoring | 사건 기록과 집계·경보 시스템입니다. |
| TTFT vs elapsed | 첫 출력까지와 전체 완료까지입니다. |
| Debug context vs secret | 문제 분석 정보와 절대 기록하면 안 되는 자격증명입니다. |

## 연결되는 개념

- 이전: [System Prompt와 Template](04-system-prompts-and-templates.md)
- 다음: [응답 캐시와 미니 프로젝트](06-response-cache-and-mini-project.md)

## 셀프 체크

- [ ] Module logger를 사용한다.
- [ ] Request ID를 레이어 간 전달한다.
- [ ] TTFT와 전체 latency를 구분한다.
- [ ] 민감정보를 log에서 제거한다.
- [ ] 결과 category를 구조화한다.

### 복습 질문 및 답변

**Q1. 사용자 질문 전체를 로그에 남기면 편하지 않나요?**

<details>
<summary>답</summary>

개인정보와 기밀이 포함될 수 있어 기본값으로 피하고, 목적·동의·보존기간을 갖춘 별도 정책이 필요합니다.

</details>

**Q2. 오류를 전부 ERROR level로 남겨야 하나요?**

<details>
<summary>답</summary>

예상된 사용자 입력 오류와 취소, 일시적 제한, 시스템 장애를 severity에 맞게 분류해야 경보가 유용합니다.

</details>

**Q3. Request ID는 provider ID와 같은가요?**

<details>
<summary>답</summary>

서비스가 발급한 ID와 provider가 반환한 ID는 별개이며 둘을 함께 기록하면 end-to-end 추적에 도움이 됩니다.

</details>

## 한 줄 정리

> 좋은 로그는 많은 문장이 아니라 요청을 재구성할 수 있는 안전한 구조화 field입니다.
