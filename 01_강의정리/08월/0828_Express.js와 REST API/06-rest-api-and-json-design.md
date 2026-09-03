# REST API와 JSON 설계

REST API의 목표는 URL 모양을 맞추는 데 그치지 않고 자원과 행위, 상태와 표현을 예측 가능한 계약으로 만드는 것입니다.

**핵심 키워드:** REST, resource, HTTP method, status code, JSON

## 핵심 내용

- URL은 동사보다 자원을 명사로 표현합니다.
- HTTP 메서드로 조회, 생성, 전체 교체, 부분 수정과 삭제 의도를 구분합니다.
- 계층 관계는 `/users/:userId/orders`처럼 필요한 범위에서 표현합니다.
- 상태 코드와 오류 JSON을 일관되게 사용합니다.
- JSON은 언어 독립적인 데이터 표현이며 객체 키는 문자열이고 값에는 함수나 undefined를 넣을 수 없습니다.

## 자원 중심 URL

| 작업 | 메서드와 경로 | 일반적인 성공 상태 |
|---|---|---|
| 목록 조회 | `GET /notes` | 200 |
| 상세 조회 | `GET /notes/:noteId` | 200 |
| 생성 | `POST /notes` | 201 |
| 부분 수정 | `PATCH /notes/:noteId` | 200 또는 204 |
| 삭제 | `DELETE /notes/:noteId` | 204 또는 정책에 따른 응답 |

`/getNotes`, `/createNote`처럼 URL에 행위를 반복하기보다 메서드와 자원 이름을 조합합니다.

## JSON 표현

```json
{
  "id": 12,
  "title": "REST 원칙",
  "tags": ["http", "api"],
  "published": true
}
```

- **목적:** 하나의 노트 자원을 클라이언트와 교환 가능한 형식으로 표현합니다.
- **흐름:** 서버 객체 선택 → 공개 필드 직렬화 → Content-Type 설정 → 전송입니다.
- **결과:** 다양한 언어의 클라이언트가 같은 데이터를 해석할 수 있습니다.
- **실무 포인트:** 내부 모델을 그대로 노출하지 말고 API 계약에 필요한 필드만 명시적으로 구성합니다.

## PUT과 PATCH

| 메서드 | 의도 | 주의점 |
|---|---|---|
| PUT | 대상 자원의 전체 표현 교체 | 누락 필드 처리 규칙 명확화 |
| PATCH | 지정한 필드만 부분 수정 | 허용 필드와 유효성 검증 |

실제 서비스에서는 문서화한 계약이 가장 중요합니다. 같은 API 안에서 수정 의미와 응답 구조를 일관되게 유지합니다.

## 계층과 필터

소유 관계를 드러낼 필요가 있을 때 `/users/3/orders` 같은 계층 경로를 사용할 수 있습니다. 너무 깊은 URL은 결합도를 높이므로 독립 자원 ID로 조회할 수 있는 경우 단순한 경로와 query filter를 검토합니다.

```http
GET /orders?userId=3&status=paid&page=2
```

## 실습

1. 게시글 CRUD의 메서드·경로·성공 상태를 설계하세요.
2. PUT과 PATCH의 차이를 설명하세요.
3. 일관된 validation 오류 JSON을 작성하세요.

<details>
<summary>답</summary>

목록 `GET /posts`, 상세 `GET /posts/:postId`, 생성 `POST /posts`, 부분 수정 `PATCH /posts/:postId`, 삭제 `DELETE /posts/:postId`로 설계할 수 있습니다. 오류는 예를 들어 `{"error":"validation_failed","fields":{"title":"required"}}`처럼 안정적인 코드를 제공합니다.

</details>

## 더 알아보기

- [요청 데이터와 응답 작성](03-request-data-and-responses.md)
- [MVC 기반 CRUD와 API 테스트](07-mvc-crud-and-api-testing.md)

## 체크리스트

- [ ] URL에 자원을 명사로 표현한다.
- [ ] HTTP 메서드와 상태를 일관되게 쓴다.
- [ ] PUT과 PATCH 계약을 구분한다.
- [ ] 내부 데이터 필드를 그대로 노출하지 않는다.
- [ ] 오류와 페이지네이션 형식을 문서화한다.

## 복습 질문 및 답변

### Q1. REST API는 반드시 JSON만 사용하나요?

<details>
<summary>답</summary>

아닙니다. REST는 특정 표현 형식에 종속되지 않습니다. JSON이 웹 API에서 널리 사용되지만 요구에 따라 다른 미디어 타입도 사용할 수 있습니다.

</details>

### Q2. URL에 동사를 넣지 않는 이유는 무엇인가요?

<details>
<summary>답</summary>

자원은 URL로, 행위는 HTTP 메서드로 나누면 인터페이스가 일관되고 예측 가능해지기 때문입니다.

</details>

### Q3. 200 상태로 모든 실패를 반환하면 어떤 문제가 있나요?

<details>
<summary>답</summary>

클라이언트, 캐시, 모니터링이 성공과 실패를 표준적으로 판단하기 어려워지고 각자 본문을 해석해야 합니다.

</details>

## 요약

REST API는 자원 URL, HTTP 메서드, 상태 코드와 표현 형식을 하나의 계약으로 설계합니다. 일관된 JSON과 오류 구조가 클라이언트와 서버의 결합을 줄입니다.
