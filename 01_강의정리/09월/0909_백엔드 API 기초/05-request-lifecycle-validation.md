# 요청 처리 흐름과 입력 검증

서버는 요청을 받자마자 업무 로직을 실행하지 않습니다. Routing, 인증, 파싱, 검증을 거쳐 안전한 값만 service에 전달하고, 결과를 일관된 응답으로 변환합니다.

**핵심 키워드:** routing, handler, validation, service layer, error handling

## 요청의 이동 경로

```text
Client → Router → Authentication → Parsing → Validation
       → Handler → Service → Repository/External API
       → Response serialization → Client
```

Middleware는 여러 endpoint에 공통으로 필요한 인증, 요청 ID, 로깅, CORS 같은 처리를 맡을 수 있습니다. Handler는 HTTP 입력을 받아 service를 호출하고 결과를 HTTP 응답으로 바꾸는 경계 역할을 합니다.

<details>
<summary>답</summary>

Routing은 method와 path가 어떤 handler로 연결될지 결정합니다. URL이 맞아도 method가 다르면 다른 handler가 선택되거나 405 응답을 받을 수 있습니다.

</details>

## 경계에서 검증하기

```python
from fastapi import FastAPI, status
from pydantic import BaseModel, Field

app = FastAPI()

class NoteCreate(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    body: str = Field(min_length=1)

@app.post("/notes", status_code=status.HTTP_201_CREATED)
def create_note(payload: NoteCreate):
    note = {"id": 1, **payload.model_dump()}
    return {"data": note}
```

형식과 타입은 schema에서, 현재 재고나 권한처럼 외부 상태가 필요한 규칙은 service에서 검사합니다. 검증 위치를 분리하면 규칙이 중복되지 않고 테스트도 쉬워집니다.

<details>
<summary>답</summary>

인증(authentication)은 사용자가 누구인지 확인하고, 인가(authorization)는 그 사용자가 해당 작업을 할 권한이 있는지 판단합니다. 일반적으로 인증 실패는 401, 권한 부족은 403으로 구분합니다.

</details>

## Handler와 Service 분리

```python
class NoteService:
    def create(self, title: str, body: str) -> dict:
        # 업무 규칙과 저장소 호출은 이 계층에서 처리한다.
        return {"id": 1, "title": title, "body": body}

service = NoteService()

@app.post("/service-notes", status_code=201)
def create_service_note(payload: NoteCreate):
    return {"data": service.create(payload.title, payload.body)}
```

Handler가 데이터베이스와 외부 API를 모두 직접 다루면 재사용과 단위 테스트가 어려워집니다. Service는 HTTP와 무관한 업무 규칙을 중심으로 유지하고, 외부 호출에는 timeout과 실패 변환 정책을 둡니다.

<details>
<summary>답</summary>

`async def`를 사용한다고 모든 작업이 자동으로 빨라지지는 않습니다. 비동기 handler 안에서 동기 I/O를 실행하면 event loop를 막을 수 있으므로 사용하는 라이브러리와 실행 방식을 함께 확인해야 합니다.

</details>

## 오류를 안정적인 계약으로 바꾸기

```json
{
  "error": {
    "code": "NOTE_NOT_FOUND",
    "message": "요청한 노트를 찾을 수 없습니다.",
    "request_id": "req_b12"
  }
}
```

| 상태 | 대표 상황 |
|---|---|
| 400 | 요청 형식이나 의미가 잘못됨 |
| 401 | 인증 정보가 없거나 유효하지 않음 |
| 403 | 인증됐지만 작업 권한이 없음 |
| 404 | 대상 리소스가 없음 |
| 409 | 현재 상태와 요청이 충돌함 |
| 422 | 구조는 읽었지만 field 검증에 실패함 |
| 500 | 예상하지 못한 서버 오류 |

외부 응답에는 비밀값, stack trace, 파일 경로를 넣지 않습니다. 상세 원인은 `request_id`와 함께 서버 로그에 기록합니다.

<details>
<summary>답</summary>

예상 가능한 도메인 오류와 예상하지 못한 시스템 오류를 나누어 처리해야 합니다. 모든 예외를 한꺼번에 잡아 200으로 반환하면 모니터링과 클라이언트 분기가 모두 어려워집니다.

</details>

## 비교: 비대한 Handler와 계층 분리

| 관점 | 비대한 Handler | Handler + Service |
|---|---|---|
| 책임 | HTTP·업무·저장이 혼합 | 경계와 업무 규칙 분리 |
| 테스트 | 전체 환경 필요 | service 단위 테스트 가능 |
| 재사용 | 다른 진입점에서 어려움 | CLI·batch에서도 재사용 가능 |
| 변경 영향 | 넓음 | 계층별로 제한 가능 |

## 실습

1. 새 노트 생성 요청의 검증 규칙 세 가지를 정하세요.
2. 인증되지 않은 요청과 권한 없는 요청의 상태 코드를 고르세요.
3. 외부 요약 서비스가 timeout일 때 handler와 service의 책임을 나누세요.

<details>
<summary>정답 보기</summary>

예: 제목 필수·100자 이하, 본문 필수, 허용된 공개 범위만 입력하도록 검증합니다. 인증 실패는 401, 권한 부족은 403입니다. Service는 timeout과 외부 오류를 도메인에 맞게 변환하고, handler는 이를 일관된 5xx 응답과 request ID로 전달할 수 있습니다.

</details>

## 셀프 체크

- [ ] 요청이 handler에 도달하는 흐름을 설명할 수 있다.
- [ ] schema 검증과 업무 규칙 검증을 구분할 수 있다.
- [ ] 인증과 인가의 차이를 설명할 수 있다.
- [ ] handler와 service의 책임을 분리할 수 있다.
- [ ] 내부 정보를 숨긴 오류 응답을 설계할 수 있다.

이전 글: [REST 리소스와 Endpoint 설계](04-rest-resource-design.md) · 다음 글: [API 클라이언트 호출과 테스트](06-api-client-testing.md)
