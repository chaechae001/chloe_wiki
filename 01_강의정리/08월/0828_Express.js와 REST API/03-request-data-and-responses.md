# 요청 데이터와 응답 작성

API의 입력은 경로, 쿼리, 본문과 헤더에 나뉘므로 값의 의미에 맞는 위치를 선택하고 검증해야 합니다.

**핵심 키워드:** params, query, body, headers, response

## 핵심 내용

- Path parameter는 특정 자원의 식별자를 경로 안에 표현합니다.
- Query string은 필터, 정렬, 페이지처럼 선택적인 조회 조건에 적합합니다.
- Request body는 생성·수정할 구조화 데이터를 전달합니다.
- Header는 콘텐츠 형식, 인증, 캐시와 같은 요청 메타데이터를 담습니다.
- 입력은 모두 신뢰하지 말고 형식과 범위, 권한을 검증한 뒤 사용합니다.

## 입력 위치 비교

| 입력 | Express 접근 | 예시 | 적합한 의미 |
|---|---|---|---|
| Path | `req.params` | `/books/17` | 특정 자원 식별 |
| Query | `req.query` | `?page=2&sort=title` | 선택적 조회 조건 |
| Body | `req.body` | JSON 객체 | 생성·수정 데이터 |
| Header | `req.get()` | `Authorization` | 요청 메타데이터 |

## 라우트 입력 처리

```javascript
app.use(express.json());

app.get("/books/:bookId", (req, res) => {
  const bookId = Number(req.params.bookId);
  if (!Number.isInteger(bookId) || bookId < 1) {
    return res.status(400).json({ error: "invalid_book_id" });
  }

  res.status(200).json({ id: bookId });
});
```

- **목적:** 경로 문자열을 정수 식별자로 검증한 뒤 JSON으로 응답합니다.
- **흐름:** 파라미터 추출 → 변환 → 검증 → 실패 또는 성공 응답입니다.
- **결과:** 잘못된 식별자는 400, 올바른 입력은 200을 받습니다.
- **실무 포인트:** 응답을 보낸 뒤에는 `return`하여 다음 코드가 다시 응답하지 않도록 합니다.

## JSON 본문 처리

```javascript
app.post("/books", (req, res) => {
  const { title } = req.body;
  if (typeof title !== "string" || title.trim() === "") {
    return res.status(400).json({ error: "title_required" });
  }

  const book = { id: 18, title: title.trim() };
  return res.status(201).json(book);
});
```

`express.json()`이 라우트보다 앞에 등록되어야 JSON 본문을 파싱할 수 있습니다. 파싱 성공이 데이터 유효성을 보장하지는 않으므로 별도 검증이 필요합니다.

## 응답 API

`res.status()`로 상태를, `res.json()`으로 JSON을 보냅니다. `res.send()`는 다양한 본문을 처리하지만 API에서는 일관된 JSON 구조가 클라이언트 구현에 유리합니다.

## 실습

1. `/products/:productId`의 양의 정수 ID를 검증하세요.
2. `?limit=20` 값을 허용 범위로 제한하세요.
3. JSON 본문에 이름이 없으면 400을 반환하는 라우트를 작성하세요.

<details>
<summary>답</summary>

```javascript
app.post("/users", (req, res) => {
  const { name } = req.body;
  if (typeof name !== "string" || !name.trim()) {
    return res.status(400).json({ error: "name_required" });
  }
  return res.status(201).json({ id: 1, name: name.trim() });
});
```

</details>

## 더 알아보기

- [Express 애플리케이션과 Router](02-express-app-and-router.md)
- [REST API와 JSON 설계](06-rest-api-and-json-design.md)

## 체크리스트

- [ ] 입력 의미에 맞는 위치를 선택한다.
- [ ] 문자열 입력을 필요한 타입으로 변환한다.
- [ ] 형식, 범위와 권한을 검증한다.
- [ ] body parser의 등록 순서를 확인한다.
- [ ] 상태와 JSON 오류 형식을 일관되게 반환한다.

## 복습 질문 및 답변

### Q1. Path parameter와 Query parameter의 차이는 무엇인가요?

<details>
<summary>답</summary>

Path parameter는 특정 자원이나 계층을 식별하는 데 주로 쓰고 Query parameter는 필터, 정렬, 페이지 같은 선택 조건에 사용합니다.

</details>

### Q2. `express.json()`을 사용하면 입력 검증이 끝나나요?

<details>
<summary>답</summary>

아닙니다. JSON 문자열을 JavaScript 값으로 파싱할 뿐 필드의 타입, 범위, 비즈니스 규칙과 권한은 별도로 검증해야 합니다.

</details>

### Q3. 응답 후 return이 유용한 이유는 무엇인가요?

<details>
<summary>답</summary>

현재 처리 흐름을 종료해 이후 코드가 실행되거나 같은 요청에 두 번째 응답을 보내는 실수를 예방합니다.

</details>

## 요약

Express 요청 데이터는 params, query, body와 headers로 나뉩니다. 위치별 의미를 지키고 입력을 검증하며 상태 코드와 JSON 구조를 일관되게 반환해야 합니다.
