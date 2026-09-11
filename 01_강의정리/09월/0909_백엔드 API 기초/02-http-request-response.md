# HTTP 요청과 응답 메시지

HTTP 통신은 클라이언트가 요청을 보내고 서버가 응답하는 한 쌍의 메시지로 이루어집니다. 각 메시지의 역할을 분리해서 읽으면 API 오류를 훨씬 빠르게 찾을 수 있습니다.

**핵심 키워드:** method, path, header, body, status code

## 요청 메시지의 네 요소

```http
POST /messages HTTP/1.1
Host: api.example.com
Authorization: Bearer [REDACTED]
Content-Type: application/json

{"text": "회의 내용을 세 줄로 요약해 주세요."}
```

- **Method**: 요청의 의도입니다.
- **Path**: 대상 리소스의 위치입니다.
- **Header**: 인증, 콘텐츠 형식 같은 부가 정보입니다.
- **Body**: 생성·변경할 데이터입니다.

GET 요청의 본문은 의미가 명확하게 표준화되어 있지 않고 일부 서버와 중간 장비가 지원하지 않습니다. 조회 조건은 보통 `?page=2&size=20`처럼 query parameter로 전달합니다.

<details>
<summary>답</summary>

URL은 리소스의 위치를, method는 그 리소스에 수행할 동작의 의미를 표현합니다. 같은 `/messages/7`이라도 GET은 조회, DELETE는 삭제를 뜻할 수 있습니다.

</details>

## Method의 의미와 성질

| Method | 대표 의미 | Safe | Idempotent |
|---|---|---:|---:|
| GET | 조회 | 예 | 예 |
| POST | 생성·처리 요청 | 아니요 | 일반적으로 아니요 |
| PUT | 전체 교체 | 아니요 | 예 |
| PATCH | 부분 수정 | 아니요 | 구현에 따라 다름 |
| DELETE | 삭제 | 아니요 | 의미상 예 |

Safe는 서버 상태를 바꾸려는 목적이 없다는 뜻이고, idempotent는 같은 요청을 여러 번 보내도 의도된 최종 상태가 같다는 뜻입니다. 네트워크 재시도 정책은 이 성질을 고려해야 합니다.

<details>
<summary>답</summary>

POST가 항상 생성만 뜻하는 것은 아닙니다. 검색 실행, 파일 변환처럼 리소스에 처리 명령을 전달할 때도 사용할 수 있습니다. 다만 endpoint 이름과 문서에서 의미가 분명해야 합니다.

</details>

## 응답 메시지 읽기

```http
HTTP/1.1 201 Created
Location: /messages/42
Content-Type: application/json

{"data": {"id": 42, "status": "queued"}}
```

응답은 상태 코드, header, body로 구성됩니다. `201 Created`는 생성 성공을 나타내며 `Location` header로 새 리소스의 위치를 알려줄 수 있습니다. `204 No Content` 응답에는 본문을 넣지 않습니다.

| 범위 | 의미 | 예시 |
|---|---|---|
| 2xx | 성공 | 200, 201, 204 |
| 3xx | 리다이렉션 | 301, 304 |
| 4xx | 요청 측 문제 | 400, 401, 403, 404, 409, 422 |
| 5xx | 서버 측 문제 | 500, 503 |

응답을 받았다는 사실과 요청이 성공했다는 사실은 다릅니다. 클라이언트는 반드시 상태 코드를 확인해야 합니다.

<details>
<summary>답</summary>

`404 Not Found`는 대상 리소스를 찾지 못했다는 뜻이고, `500 Internal Server Error`는 서버가 예상하지 못한 문제를 만났다는 뜻입니다. 같은 실패라도 책임 영역과 대응 방법이 다릅니다.

</details>

## Content-Type과 Accept

`Content-Type`은 현재 메시지 본문의 형식을 알립니다. `Accept`는 클라이언트가 받고 싶은 응답 형식을 제안합니다. JSON endpoint에 다른 형식의 본문을 보내면 서버는 `415 Unsupported Media Type`으로 거절할 수 있습니다.

```http
Accept: application/json
Content-Type: application/json; charset=utf-8
```

<details>
<summary>답</summary>

JSON처럼 보여도 `Content-Type`이 맞지 않으면 프레임워크가 본문 파싱을 거부할 수 있습니다. 본문의 내용과 header를 함께 확인해야 합니다.

</details>

## 비교: 요청과 응답

| 구분 | Request | Response |
|---|---|---|
| 시작 정보 | method + target | status code |
| header | 조건·인증·본문 형식 | 결과 메타데이터·본문 형식 |
| body | 서버에 전달할 데이터 | 처리 결과 또는 오류 정보 |

## 실습

1. 회원 목록 조회 요청에서 method, path, query parameter를 설계해 보세요.
2. 생성 성공·검증 실패·서버 장애에 알맞은 상태 코드를 고르세요.
3. JSON 응답을 원한다는 요청 header를 작성하세요.

<details>
<summary>정답 보기</summary>

예: `GET /members?page=1&size=20`, 생성 성공은 201, 검증 실패는 422, 예상하지 못한 서버 장애는 500을 사용할 수 있습니다. JSON 응답 선호는 `Accept: application/json`으로 표현합니다.

</details>

## 셀프 체크

- [ ] 요청의 method, path, header, body를 구분할 수 있다.
- [ ] safe와 idempotent의 차이를 설명할 수 있다.
- [ ] 성공과 실패 상태 코드의 범위를 해석할 수 있다.
- [ ] `Content-Type`과 `Accept`의 역할을 구분할 수 있다.
- [ ] 응답 수신과 성공을 구별해 처리할 수 있다.

다음 글: [JSON과 데이터 직렬화](03-json-serialization.md)
