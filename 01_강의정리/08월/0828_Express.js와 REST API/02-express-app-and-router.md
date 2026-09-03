# Express 애플리케이션과 Router

Express.js는 Node.js의 HTTP 처리를 라우트와 미들웨어 단위로 조립하게 해 주는 가벼운 웹 프레임워크입니다.

**핵심 키워드:** Express, app, Router, route handler, listen

## 핵심 내용

- Express 애플리케이션 객체는 미들웨어, 라우트와 서버 설정을 연결합니다.
- 라우팅은 HTTP 메서드와 경로를 처리 함수에 매핑합니다.
- `Express.Router`는 관련 라우트를 별도 모듈로 묶는 작은 애플리케이션처럼 동작합니다.
- `app.use`로 공통 미들웨어나 Router를 특정 경로에 마운트합니다.
- 서버 시작 코드와 애플리케이션 구성을 분리하면 테스트가 쉬워집니다.

## 최소 애플리케이션

```javascript
const express = require("express");

const app = express();

app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});

app.listen(3000, () => {
  console.log("server started");
});
```

- **목적:** 상태 확인 엔드포인트를 제공하는 HTTP 서버를 시작합니다.
- **흐름:** 앱 생성 → GET 라우트 등록 → 포트 수신 → 요청 처리입니다.
- **결과:** `GET /health`에 JSON 응답을 반환합니다.
- **실무 포인트:** 포트와 비밀 설정은 코드에 고정하지 않고 환경에 따라 주입합니다.

## Router로 책임 나누기

```javascript
// routes/articles.js
const express = require("express");
const router = express.Router();

router.get("/", listArticles);
router.get("/:articleId", getArticle);
router.post("/", createArticle);

module.exports = router;
```

```javascript
const articleRouter = require("./routes/articles");
app.use("/articles", articleRouter);
```

`router.get("/")`은 마운트 경로와 합쳐져 `GET /articles`가 됩니다. Router 파일에는 HTTP 연결 책임을 두고 복잡한 비즈니스 로직은 별도 서비스로 분리합니다.

## app 라우팅과 Router

| 방식 | 적합한 상황 | 주의점 |
|---|---|---|
| `app.get` | 작은 앱, 공통 엔드포인트 | 한 파일이 빠르게 커질 수 있음 |
| `router.get` | 기능·자원별 라우트 묶음 | 마운트 경로와 내부 경로를 함께 읽어야 함 |

## 시작 방식 선택

직접 `npm init` 후 필요한 패키지를 추가하면 구조를 명확히 이해할 수 있습니다. 생성 도구는 빠르게 틀을 만들지만 불필요한 파일과 오래된 관례가 포함될 수 있으므로 생성 결과를 검토합니다.

## 실습

1. `/books` 목록과 `/books/:bookId` 상세 Router를 작성하세요.
2. Router를 `/api/books`에 마운트했을 때 실제 URL을 적으세요.
3. app 구성과 `listen`을 분리하면 어떤 테스트가 쉬워지는지 설명하세요.

<details>
<summary>답</summary>

```javascript
const router = require("express").Router();
router.get("/", listBooks);
router.get("/:bookId", getBook);
app.use("/api/books", router);
```

실제 경로는 `GET /api/books`와 `GET /api/books/:bookId`입니다. app을 export하면 네트워크 포트를 직접 열지 않고 요청 테스트 도구로 검증하기 쉽습니다.

</details>

## 더 알아보기

- [요청 데이터와 응답 작성](03-request-data-and-responses.md)
- [미들웨어 파이프라인](04-middleware-pipeline.md)

## 체크리스트

- [ ] app 객체의 역할을 설명한다.
- [ ] 메서드와 경로로 라우트를 등록한다.
- [ ] 자원별 Router를 분리한다.
- [ ] 마운트 경로와 내부 경로를 결합해 읽는다.
- [ ] 앱 구성과 서버 시작을 분리한다.

## 복습 질문 및 답변

### Q1. Router는 별도의 서버인가요?

<details>
<summary>답</summary>

아닙니다. 관련 라우트와 미들웨어를 묶어 기존 Express 앱에 마운트하는 모듈형 라우팅 객체입니다.

</details>

### Q2. `app.use("/articles", router)`의 의미는 무엇인가요?

<details>
<summary>답</summary>

`/articles`로 시작하는 요청을 해당 Router 스택으로 전달하며 Router 내부 경로 앞에 마운트 경로가 붙습니다.

</details>

### Q3. 생성 도구를 사용하면 구조를 검토하지 않아도 되나요?

<details>
<summary>답</summary>

아닙니다. 생성된 의존성, 폴더, 템플릿 엔진, 오류 처리와 보안 설정이 프로젝트 요구에 맞는지 확인해야 합니다.

</details>

## 요약

Express 앱은 라우트와 미들웨어를 조립하는 중심 객체이고 Router는 기능별 경계를 만듭니다. 경로 계층과 실행 책임을 분리하면 확장과 테스트가 쉬워집니다.
