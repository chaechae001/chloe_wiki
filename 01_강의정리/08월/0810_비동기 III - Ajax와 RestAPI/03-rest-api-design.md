# 자원 중심의 REST API 설계

좋은 API 주소는 기능 이름을 나열하지 않습니다. 무엇을 다루는지 주소로 표현하고, 무엇을 할지는 HTTP 메서드로 구분합니다.

## 핵심 키워드

`REST` · `자원` · `URI` · `HTTP 메서드` · `표현` · `RESTful`

## 핵심 요약

- REST는 HTTP를 활용하는 자원 중심의 아키텍처 스타일입니다.
- URI는 자원을 식별하고 메서드는 자원에 대한 행위를 표현합니다.
- 요청과 응답 본문의 JSON은 자원의 한 표현입니다.
- 일관된 규칙은 API를 예측하고 유지보수하기 쉽게 만듭니다.

## 1. REST의 세 요소

REST API는 자원, 행위, 표현의 조합으로 읽을 수 있습니다.

```text
PATCH /tasks/42
Content-Type: application/json

{"done":true}
```

- `/tasks/42`는 42번 할 일이라는 **자원**입니다.
- `PATCH`는 일부를 수정한다는 **행위**입니다.
- JSON 본문은 변경할 상태의 **표현**입니다.

주소에는 `updateTask` 같은 동사를 넣기보다 `/tasks/42`처럼 명사형 자원을 두고 행위를 메서드에 맡기는 편이 일관됩니다.

## 2. URI와 URL

URI는 자원을 식별하는 문자열을 넓게 가리킵니다. URL은 그중 자원의 위치와 접근 방법을 포함한 주소입니다. 실무 대화에서는 API 경로나 URL이라는 표현을 자주 사용하지만, REST의 핵심은 자원을 안정적으로 식별한다는 점입니다.

| 요청 | 의미 |
|---|---|
| `GET /tasks` | 할 일 목록 조회 |
| `POST /tasks` | 새 할 일 생성 |
| `GET /tasks/42` | 42번 할 일 조회 |
| `PATCH /tasks/42` | 42번 할 일 일부 수정 |
| `DELETE /tasks/42` | 42번 할 일 삭제 |
| `GET /tasks/42/comments` | 42번 할 일의 댓글 목록 조회 |

## 3. REST 제약의 실용적 의미

REST는 단순한 주소 작명법보다 넓은 설계 원칙입니다.

- **클라이언트–서버 분리**: 화면과 데이터 처리 책임을 나눕니다.
- **무상태**: 각 요청이 처리에 필요한 문맥을 담습니다.
- **캐시 가능성**: 재사용할 수 있는 응답인지 명확히 표현합니다.
- **일관된 인터페이스**: 자원을 식별하고 조작하는 규칙을 통일합니다.
- **계층화**: 클라이언트가 중간 서버의 존재를 몰라도 동작하게 합니다.
- **코드 온 디맨드(선택 사항)**: 필요하면 서버가 실행 가능한 코드를 전달할 수 있습니다.

모든 API가 완벽한 REST를 구현하는 것은 아닙니다. 중요한 것은 팀이 정한 규칙을 일관되게 적용하고 문서화하는 것입니다.

## 대표 코드: 요청 설명 만들기

### 목적

자원과 행위를 분리해 CRUD 요청을 예측 가능한 형태로 구성합니다.

```javascript
function buildTaskRequest(action, id, payload) {
  const base = "/api/tasks";

  const requests = {
    list: { method: "GET", url: base },
    create: { method: "POST", url: base, body: payload },
    read: { method: "GET", url: `${base}/${id}` },
    update: { method: "PATCH", url: `${base}/${id}`, body: payload },
    remove: { method: "DELETE", url: `${base}/${id}` },
  };

  return requests[action];
}

console.log(buildTaskRequest("update", 42, { done: true }));
```

### 흐름과 결과

1. 공통 자원 경로를 한 곳에 둡니다.
2. 동작에 따라 메서드와 주소를 조합합니다.
3. 생성과 수정에는 필요한 표현을 본문으로 추가합니다.
4. 수정 요청은 `{ method: "PATCH", url: "/api/tasks/42", body: { done: true } }`가 됩니다.

### 실무 연결

프론트엔드의 API 모듈을 자원별로 나누면 화면 컴포넌트가 URL 문자열과 메서드 규칙을 반복해서 알 필요가 없습니다.

## 직접 해보기

특정 할 일의 댓글을 생성하는 요청을 메서드와 경로로 표현해 보세요.

<details>
<summary>답</summary>

`POST /api/tasks/42/comments`처럼 표현할 수 있습니다. 댓글 모음이라는 하위 자원에 새 표현을 생성한다는 의미입니다.

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| URI vs URL | URI는 식별자 전체 개념이고 URL은 위치와 접근 방법을 포함한 식별자입니다. |
| 자원 vs 표현 | 자원은 관리 대상 개념이고 표현은 특정 시점의 상태를 JSON 등으로 나타낸 데이터입니다. |
| REST vs RESTful API | REST는 설계 스타일이고 RESTful은 그 원칙을 일관되게 적용한 정도를 표현합니다. |

## 연결되는 개념

- 메서드와 본문 구조는 [HTTP 요청과 응답 메시지](02-http-messages.md)에서 확인할 수 있습니다.
- 설계한 API를 호출하는 법은 [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md)에서 이어집니다.
- 호출 코드를 재사용하는 법은 [Axios 요청 패턴](05-axios-request-patterns.md)에서 다룹니다.

## 셀프 체크

- [ ] REST의 자원·행위·표현을 구분할 수 있다.
- [ ] CRUD 동작을 메서드와 경로로 표현할 수 있다.
- [ ] URI에 명사를 사용하는 이유를 설명할 수 있다.

## 복습 질문 및 답변

### Q1. `/getTasks`보다 `/tasks`가 자원 중심 설계에 가까운 이유는?

<details>
<summary>답</summary>

주소는 대상 자원만 나타내고 조회 행위는 `GET` 메서드가 담당하므로 역할이 분리되기 때문입니다.

</details>

### Q2. `PATCH /tasks/3`의 세 요소를 설명하면?

<details>
<summary>답</summary>

`/tasks/3`은 자원, `PATCH`는 일부 수정 행위, 요청 본문의 JSON은 변경할 상태의 표현입니다.

</details>

### Q3. 무상태 API에서 인증된 요청은 어떻게 사용자 문맥을 전달하는가?

<details>
<summary>답</summary>

각 요청에 인증 토큰이나 쿠키 같은 식별 정보를 함께 보내 서버가 그 요청을 독립적으로 처리하게 합니다.

</details>

## 한 줄 정리

> REST API는 URI로 자원을 가리키고 HTTP 메서드로 행위를 표현해 예측 가능한 통신 규칙을 만듭니다.
