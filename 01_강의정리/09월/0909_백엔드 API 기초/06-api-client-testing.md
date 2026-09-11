# API 클라이언트 호출과 테스트

API client는 요청을 보내는 것만큼 실패를 정확히 분류하는 것이 중요합니다. Timeout, 상태 코드, 콘텐츠 형식을 확인하고 성공·실패 계약을 자동 테스트해야 안정적인 연동이 가능합니다.

**핵심 키워드:** timeout, status code, exception, contract test, observability

## 안전하게 호출하기

```python
import requests

def fetch_note(note_id: int) -> dict:
    response = requests.get(
        f"https://api.example.com/notes/{note_id}",
        headers={"Accept": "application/json"},
        timeout=(3, 10),
    )
    response.raise_for_status()
    return response.json()
```

Timeout을 생략하면 장애 상황에서 호출이 오래 대기할 수 있습니다. 연결 timeout과 응답 읽기 timeout을 구분하고, session을 사용하면 연결 재사용과 공통 header 관리가 쉬워집니다.

<details>
<summary>답</summary>

재시도는 모든 요청에 적용하면 안 됩니다. GET처럼 idempotent한 요청이나 idempotency key로 보호된 요청을 대상으로, 지수 backoff와 최대 횟수를 정해 일시적 오류에만 적용합니다.

</details>

## 실패 응답도 데이터다

```python
def decode_response(response: requests.Response) -> dict:
    content_type = response.headers.get("Content-Type", "")
    if "application/json" not in content_type:
        return {"status": response.status_code, "raw": response.text[:200]}
    return {"status": response.status_code, "payload": response.json()}
```

`raise_for_status()`는 편리하지만, 호출 전에 오류 body를 로깅하거나 도메인 오류로 변환해야 할 수 있습니다. HTML 오류 페이지나 빈 body가 올 수 있으므로 무조건 `.json()`을 호출하지 않습니다.

<details>
<summary>답</summary>

Connection error는 서버에서 HTTP 응답을 받지 못한 전송 계층 문제이고, 500은 서버가 HTTP 응답으로 실패를 알린 경우입니다. 관측 지표와 재시도 판단이 달라집니다.

</details>

## 테스트 층 나누기

| 테스트 | 검증 대상 | 특징 |
|---|---|---|
| Unit | service 함수·규칙 | 빠르고 외부 의존성이 적음 |
| Integration | router·DB·외부 adapter | 계층 간 연결 확인 |
| Contract | 요청·응답 schema와 상태 | client-server 호환성 확인 |
| End-to-end | 실제 사용자 흐름 | 현실적이지만 느리고 비용이 큼 |

성공 사례만 검사하면 실제 장애에 취약합니다. 필수 field 누락, 잘못된 타입, 인증 실패, 권한 부족, 없는 리소스, 중복 요청, timeout도 테스트합니다.

<details>
<summary>답</summary>

Mock은 빠르고 실패 조건을 만들기 쉽지만 실제 네트워크·직렬화 문제를 놓칠 수 있습니다. 핵심 경로에는 실제 구성 요소를 연결한 integration test를 함께 둡니다.

</details>

## 관측 가능성과 보안

요청마다 correlation ID를 부여하면 client 오류와 서버 로그를 연결할 수 있습니다. 로그에는 method, route pattern, status, latency를 남기되 token과 개인정보는 마스킹합니다. 알림은 단일 500보다 오류율과 지연 시간의 지속적 변화를 기준으로 설계합니다.

<details>
<summary>답</summary>

URL 전체를 그대로 기록하면 query string의 민감 정보가 로그에 남을 수 있습니다. Route pattern과 허용된 query key만 기록하고 secret, token, 개인정보는 제거합니다.

</details>

## 비교: 수동 확인과 자동 테스트

| 관점 | 수동 호출 | 자동 테스트 |
|---|---|---|
| 탐색 | 빠름 | 준비 필요 |
| 반복성 | 사람에 따라 다름 | 동일 조건 재현 |
| 회귀 발견 | 놓치기 쉬움 | 변경 때마다 확인 가능 |
| 권장 용도 | 초기 탐색·디버깅 | 계약·업무 규칙 보호 |

## 실습

1. 목록 조회 client에 timeout과 query parameter를 추가하세요.
2. JSON이 아닌 오류 응답을 안전하게 기록하는 전략을 설명하세요.
3. 노트 생성 API의 성공·실패 테스트 사례를 각각 두 개 작성하세요.

<details>
<summary>정답 보기</summary>

예: `requests.get(url, params={"page": 1}, timeout=(3, 10))`. 콘텐츠 형식을 확인한 뒤 JSON이 아니면 길이를 제한한 text와 status만 기록합니다. 성공은 201과 응답 schema, 실패는 빈 제목 422와 인증 누락 401을 검사할 수 있습니다.

</details>

## 셀프 체크

- [ ] HTTP client 호출에 timeout을 설정할 수 있다.
- [ ] 전송 오류와 HTTP 실패 응답을 구분할 수 있다.
- [ ] 응답 콘텐츠 형식을 확인한 뒤 파싱할 수 있다.
- [ ] unit, integration, contract, E2E 테스트를 구분할 수 있다.
- [ ] 민감 정보를 제외한 관측 로그를 설계할 수 있다.

이전 글: [요청 처리 흐름과 입력 검증](05-request-lifecycle-validation.md) · 전체 보기: [OVERVIEW](OVERVIEW.md)
