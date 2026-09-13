# 오류 처리·재시도·관측성

> 외부 LLM 호출 실패를 의미 있는 서비스 오류로 변환하되 내부 비밀과 공급자 세부사항은 노출하지 않습니다.

`timeout` · `retry` · `rate limit` · `request ID` · `HTTPException`

## 핵심요약

- 입력 오류, 인증 오류, 사용량 제한, 공급자 장애를 구분합니다.
- 원문 예외는 서버 로그에 제한적으로 남기고 사용자에게는 안정적인 오류 code를 줍니다.
- 재시도는 일시적 실패에만 제한하고 backoff와 최대 횟수를 둡니다.
- 요청별 timeout을 명시해 worker가 무한 대기하지 않게 합니다.
- 공급자 request ID와 서비스 request ID를 함께 기록하면 추적이 쉬워집니다.

## 1. 오류 분류

| 상황 | 서비스 응답 예시 | 재시도 |
|---|---:|---|
| 잘못된 사용자 입력 | 400/422 | 하지 않음 |
| 서비스 인증 구성 오류 | 502 또는 내부 장애 | 키 교체 후 |
| 공급자 rate limit | 429/503 정책에 따라 | 대기 후 제한적으로 |
| 공급자 timeout | 504 | 제한적으로 |
| 알 수 없는 upstream 오류 | 502 | 조건부 |

공급자 인증 실패를 그대로 401로 내리면 클라이언트 인증 실패로 오해할 수 있습니다. 내 서비스 사용자의 인증과 서버가 가진 공급자 키 오류를 구분해 설계합니다.

<details>
<summary>답</summary>

내 서버의 공급자 키가 잘못된 경우는 사용자 자격 증명 문제가 아니라 서버 구성 문제이므로 외부 계약에서 별도 내부 오류 code로 다루는 편이 명확합니다.

</details>

## 2. 좁은 try 범위

```python
try:
    reply = generate_reply(message)
except ProviderRateLimitError:
    raise HTTPException(429, "temporarily rate limited")
except ProviderTimeoutError:
    raise HTTPException(504, "model request timed out")
except ProviderError:
    raise HTTPException(502, "model provider failed")
```

응답 파싱과 내 코드의 버그까지 모두 같은 502로 숨기지 않도록 외부 호출 경계를 좁게 감쌉니다. 실제 예외 class는 설치된 SDK 버전의 공식 문서를 확인합니다.

## 3. 로그에 남길 것

서비스 request ID, endpoint, latency, 결과 category, 모델 별칭, 토큰 사용량을 구조화해 남깁니다. 입력 전문, 출력 전문, API key, Authorization header는 기본 로그에서 제외합니다. OpenAI 응답의 request ID는 지원 문의와 장애 추적에 유용합니다.

## 코드로 보기 — 안전한 오류 응답

### 코드 목적

사용자 메시지와 운영 로그의 정보 수준을 분리합니다.

### 코드 흐름

1. 요청 ID를 만듭니다.
2. 외부 호출 시간을 측정합니다.
3. 예외를 category로 매핑합니다.
4. 내부 로그와 공개 응답을 따로 만듭니다.

### 실행 결과 해석

사용자는 안정적인 오류 code로 대응하고 운영자는 request ID로 상세 원인을 추적합니다.

### 실무 연결

Retry budget과 circuit breaker로 장애 중 중복 요청 폭증을 막을 수 있습니다.

## 직접 해보기

1. Timeout과 rate limit을 다른 상태로 매핑하세요.
2. 로그 금지 항목 네 가지를 적으세요.
3. 최대 2회 지수 backoff 정책을 설계하세요.

<details>
<summary>정답 보기</summary>

Timeout은 504, rate limit은 계약에 따라 429 등으로 구분할 수 있습니다. 키·인증 header·입력 전문·출력 전문은 기본 로그에서 제외하고, 재시도는 1초와 2초처럼 상한을 둡니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| client 401 vs provider auth failure | 사용자 인증 실패와 서버의 외부 자격 증명 오류입니다. |
| retryable vs permanent | 일시적 과부하·timeout과 잘못된 입력·권한 오류입니다. |
| service ID vs provider ID | 내 요청 추적 값과 외부 API가 발급한 추적 값입니다. |

## 연결되는 개념

- 이전: [FastAPI와 LLM 연결](04-fastapi-llm-integration.md)
- 다음: [계약 테스트와 운영 점검](06-contract-testing-and-operations.md)

## 셀프 체크

- [ ] 오류 원인을 category로 분류한다.
- [ ] 공개 오류와 내부 로그를 분리한다.
- [ ] Timeout을 명시한다.
- [ ] 영구 오류는 재시도하지 않는다.
- [ ] 두 종류의 request ID를 활용한다.

### 복습 질문 및 답변

**Q1. 모든 예외를 500으로 반환하면 왜 부족한가요?**

<details>
<summary>답</summary>

클라이언트가 입력 수정, 잠시 후 재시도, 운영자 문의 중 올바른 행동을 선택할 수 없습니다.

</details>

**Q2. 원문 예외를 응답 detail에 넣어도 되나요?**

<details>
<summary>답</summary>

키, URL, 내부 구조가 섞일 수 있으므로 외부에는 안정적인 메시지만 제공해야 합니다.

</details>

**Q3. 재시도 횟수가 많을수록 안정적인가요?**

<details>
<summary>답</summary>

아닙니다. 장애와 rate limit을 악화시키고 지연·비용을 늘릴 수 있어 제한과 backoff가 필요합니다.

</details>

## 한 줄 정리

> 실패를 분류하고 제한적으로 재시도하며, 비밀 없는 구조화 로그로 요청을 추적합니다.
