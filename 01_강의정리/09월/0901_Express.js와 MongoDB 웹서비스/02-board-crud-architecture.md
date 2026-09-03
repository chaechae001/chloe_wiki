# 게시판 CRUD 아키텍처

게시판은 HTTP 요청, 입력 검증, 업무 규칙, MongoDB 작업과 응답을 분리하면 기능이 늘어도 변경 범위를 통제할 수 있습니다.

**핵심 키워드:** CRUD, router, controller, service, Mongoose

## 계층별 책임

| 계층 | 책임 |
|---|---|
| Router | URL과 HTTP 메서드 연결 |
| Controller | 요청 해석과 응답 선택 |
| Service | 권한과 업무 규칙 |
| Model | 스키마 검증과 데이터 접근 |

## 생성과 조회

```javascript
router.post("/articles", requireLogin, asyncHandler(async (req, res) => {
  const input = parseArticleInput(req.body);
  const article = await Article.create({ ...input, author: req.user.id });
  res.redirect(`/articles/${article.id}`);
}));

router.get("/articles/:id", asyncHandler(async (req, res) => {
  const article = await Article.findById(req.params.id).populate("author", "name");
  if (!article) throw new HttpError(404, "글을 찾을 수 없습니다.");
  res.render("articles/detail", { article });
}));
```

URL의 식별자는 형식을 검증하고, 응답에 필요한 작성자 필드만 조회합니다. 문서 전체를 무조건 노출하지 않습니다.

## 수정과 삭제

수정·삭제는 먼저 대상을 조회한 뒤 현재 사용자가 작성자인지 확인합니다. 클라이언트가 보낸 `author` 값은 신뢰하지 않습니다. 업데이트에는 허용 필드 목록과 `runValidators: true`를 사용하고, 삭제 후 연결된 댓글이나 파일의 처리 정책도 정합니다.

## HTTP 메서드

HTML 폼은 기본적으로 GET과 POST만 지원합니다. 메서드 오버라이드나 작은 `fetch` 요청으로 PATCH·DELETE를 표현할 수 있지만, 서버는 CSRF 방어와 권한 검사를 반드시 수행해야 합니다.

## 실습

1. 제목과 본문만 허용하는 입력 정규화 함수를 설계하세요.
2. 존재하지 않는 글과 권한 없는 수정의 상태 코드를 정하세요.
3. 삭제 시 댓글 처리 정책을 적으세요.

<details>
<summary>답</summary>

허용 필드를 명시적으로 추출하고 길이·형식을 검증합니다. 대상 없음은 404, 인증됨에도 권한 없음은 403이 적절합니다. 댓글은 함께 삭제하거나 별도 보존하되 정책과 구현이 일치해야 합니다.

</details>

## 체크리스트

- [ ] 요청·업무·데이터 책임을 분리한다.
- [ ] 식별자와 본문을 검증한다.
- [ ] 허용 필드만 저장한다.
- [ ] 수정·삭제 전에 소유권을 확인한다.
- [ ] 오류를 일관된 처리기로 전달한다.

## 복습 질문 및 답변

### Q1. 생성 후 상세 URL로 리다이렉트하는 이유는 무엇인가요?

<details>
<summary>답</summary>

새로고침으로 POST가 반복되는 것을 줄이고, 생성된 자원의 고유 URL을 사용자에게 제공합니다.

</details>

### Q2. 화면에서 수정 버튼을 숨기면 권한 검사가 끝난 것인가요?

<details>
<summary>답</summary>

아닙니다. 사용자는 직접 요청을 만들 수 있으므로 서버에서 소유권을 다시 확인해야 합니다.

</details>

### Q3. `findByIdAndUpdate`에서 검증 옵션이 중요한 이유는 무엇인가요?

<details>
<summary>답</summary>

업데이트 경로에서도 스키마 규칙을 적용해 잘못된 데이터가 저장되는 것을 막기 위해서입니다.

</details>

## 요약

안전한 CRUD는 메서드 연결보다 입력, 존재 여부, 권한과 오류 흐름을 일관되게 설계하는 작업입니다.
