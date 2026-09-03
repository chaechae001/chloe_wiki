# Pug로 서버 렌더링 구성하기

Express와 Pug를 연결하면 서버가 데이터와 HTML 구조를 결합해 완성된 화면을 응답합니다. 공통 레이아웃과 재사용 가능한 조각을 분리하는 것이 핵심입니다.

**핵심 키워드:** template engine, Pug, SSR, layout, mixin

## 핵심 내용

- `view engine`과 `views` 경로를 애플리케이션 시작 시 설정합니다.
- 라우터는 조회 결과를 `res.render()`의 지역 변수로 전달합니다.
- `extends`와 `block`으로 공통 골격을, `include`와 `mixin`으로 반복 UI를 재사용합니다.
- 템플릿에는 표현 로직만 두고 데이터 조회와 업무 규칙은 서비스 계층에 둡니다.

## 요청에서 화면까지

```javascript
app.set("view engine", "pug");
app.set("views", path.join(process.cwd(), "views"));

router.get("/articles", async (req, res) => {
  const articles = await articleService.list();
  res.render("articles/list", { pageTitle: "글 목록", articles });
});
```

```pug
extends ../layout

block content
  h1= pageTitle
  each article in articles
    a(href=`/articles/${article.id}`)= article.title
```

`=`는 값을 이스케이프해 출력합니다. 검증되지 않은 HTML을 그대로 렌더링하는 방식은 스크립트 삽입 위험이 있으므로 피합니다.

## 전역 값과 지역 값

`app.locals`는 사이트 이름처럼 모든 화면에 필요한 값에 적합합니다. 요청별 사용자나 알림은 `res.locals`에 저장합니다. 전역 객체에 요청별 데이터를 넣으면 사용자 간 값이 섞일 수 있습니다.

## 실습

1. 목록과 상세 화면이 공유할 레이아웃을 설계하세요.
2. 반복되는 카드 UI를 mixin으로 분리하세요.
3. 조회 실패 시 오류 미들웨어로 전달하세요.

<details>
<summary>답</summary>

공통 `layout.pug`에 헤더와 `block content`를 만들고, 카드 mixin에는 필요한 값만 인자로 전달합니다. 비동기 라우터의 오류는 `next(error)` 또는 공통 래퍼로 중앙 오류 처리기에 보냅니다.

</details>

## 체크리스트

- [ ] 템플릿 경로와 엔진을 한 번만 설정한다.
- [ ] 데이터 조회를 템플릿에서 수행하지 않는다.
- [ ] 공통 레이아웃과 반복 UI를 분리한다.
- [ ] 요청별 값은 `res.locals`를 사용한다.
- [ ] 사용자 입력을 안전하게 출력한다.

## 복습 질문 및 답변

### Q1. SSR의 장점은 무엇인가요?

<details>
<summary>답</summary>

브라우저가 즉시 표시할 HTML을 받고, 서버의 권한·데이터 처리 결과를 한 번에 화면으로 구성할 수 있습니다.

</details>

### Q2. `app.locals`와 `res.locals`의 차이는 무엇인가요?

<details>
<summary>답</summary>

전자는 애플리케이션 전체에서 공유되고 후자는 현재 요청에만 유지됩니다.

</details>

### Q3. 템플릿에 업무 규칙을 넣으면 왜 불리한가요?

<details>
<summary>답</summary>

화면과 규칙이 강하게 결합되어 테스트와 재사용이 어려워지고 API 같은 다른 전달 방식에서 같은 규칙을 쓰기 힘들어집니다.

</details>

## 요약

Pug는 화면 표현에 집중시키고, 라우터·서비스가 준비한 데이터를 안전하게 렌더링하도록 구성합니다.
