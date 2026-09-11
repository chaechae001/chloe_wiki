# REST 리소스와 Endpoint 설계

REST API는 서버의 기능 이름보다 사용자가 다루는 리소스를 중심으로 주소를 설계합니다. 예측 가능한 path와 HTTP method 조합은 문서가 없어도 API의 의도를 짐작하게 합니다.

**핵심 키워드:** resource, endpoint, path parameter, query parameter, CRUD

## 리소스를 명사로 표현하기

```text
GET    /projects
POST   /projects
GET    /projects/17
PATCH  /projects/17
DELETE /projects/17
```

`/getProjects`나 `/deleteProject`처럼 동사를 path에 반복하기보다 복수형 명사와 method를 조합합니다. 단, 단순 CRUD로 표현하기 어려운 처리 작업은 `/reports/17/export`처럼 하위 리소스나 명확한 action을 사용할 수 있습니다.

<details>
<summary>답</summary>

Endpoint는 특정 API 기능에 접근하는 method와 URL의 조합입니다. 같은 URL도 method가 다르면 서로 다른 endpoint로 봅니다.

</details>

## Path와 query parameter

Path parameter는 특정 리소스를 식별하고, query parameter는 목록의 필터·정렬·페이지 조건을 표현하는 데 적합합니다.

```text
GET /articles/42
GET /articles?author=7&sort=-created_at&page=2
```

| 구분 | Path parameter | Query parameter |
|---|---|---|
| 목적 | 리소스 식별 | 필터·정렬·페이지·선택 옵션 |
| 예시 | `/articles/42` | `?status=published` |
| 생략 가능성 | 보통 필수 | 보통 선택 |

<details>
<summary>답</summary>

`/users/3/orders/9` 같은 중첩 path는 소유 관계를 드러내지만 너무 깊어지면 재사용과 탐색이 어려워집니다. 관계 확인이 핵심이 아니라면 `/orders/9`처럼 직접 접근하는 endpoint도 제공합니다.

</details>

## 응답 상태와 비동기 작업

생성이 즉시 끝나면 `201 Created`, 성공했지만 돌려줄 본문이 없으면 `204 No Content`를 사용할 수 있습니다. 오래 걸리는 작업을 queue에 넣었다면 `202 Accepted`와 함께 작업 상태를 조회할 URL을 제공하는 방식이 유용합니다.

```json
{
  "data": {
    "job_id": "job_7f2",
    "status": "queued",
    "status_url": "/jobs/job_7f2"
  }
}
```

<details>
<summary>답</summary>

`202 Accepted`는 처리가 완료됐다는 뜻이 아니라 요청을 접수했다는 뜻입니다. 클라이언트가 완료 여부를 확인할 수 있도록 상태 endpoint나 callback 규칙을 함께 설계해야 합니다.

</details>

## 일관된 응답과 버전 관리

성공과 실패 응답의 구조를 일관되게 유지하면 클라이언트 구현이 단순해집니다. 목록에는 `items`와 pagination 정보를 분리하고, 오류에는 안정적인 오류 code를 제공합니다.

호환되지 않는 변경이 필요하다면 `/v2/projects` 같은 URL 버전이나 media type 버전을 검토합니다. 새 field 추가처럼 기존 클라이언트가 무시할 수 있는 변경은 꼭 새 버전을 요구하지 않습니다.

<details>
<summary>답</summary>

페이지 번호 방식은 이해하기 쉽지만 데이터가 자주 추가되면 중복·누락이 생길 수 있습니다. cursor 방식은 큰 데이터와 실시간 변화에 더 안정적이지만 구현과 디버깅이 조금 복잡합니다.

</details>

## 비교: RPC식 path와 REST식 path

| 관점 | RPC식 예시 | REST식 예시 |
|---|---|---|
| 조회 | `/getProject?id=17` | `GET /projects/17` |
| 생성 | `/createProject` | `POST /projects` |
| 삭제 | `/deleteProject?id=17` | `DELETE /projects/17` |
| 장점 | 동작을 직접 표현 | 주소와 method 규칙이 일관됨 |

## 실습

1. 댓글 목록 조회와 댓글 한 건 삭제 endpoint를 설계하세요.
2. 완료까지 시간이 걸리는 영상 변환 요청의 응답을 설계하세요.
3. 게시글 목록의 태그 필터와 cursor pagination URL을 작성하세요.

<details>
<summary>정답 보기</summary>

예: `GET /articles/10/comments`, `DELETE /comments/83`. 영상 변환은 `POST /video-jobs`에 202를 반환하고 `/video-jobs/{id}`를 제공합니다. 목록 조회는 `GET /articles?tag=api&cursor=eyJpZCI6NDJ9`처럼 표현할 수 있습니다.

</details>

## 셀프 체크

- [ ] 리소스 중심의 path를 설계할 수 있다.
- [ ] path parameter와 query parameter를 구분할 수 있다.
- [ ] CRUD와 HTTP method를 자연스럽게 연결할 수 있다.
- [ ] 201, 202, 204를 상황에 맞게 선택할 수 있다.
- [ ] pagination과 버전 관리의 선택지를 설명할 수 있다.

이전 글: [JSON과 데이터 직렬화](03-json-serialization.md) · 다음 글: [요청 처리 흐름과 입력 검증](05-request-lifecycle-validation.md)
