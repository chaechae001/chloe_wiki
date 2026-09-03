# 오류 처리와 404

오류 처리는 실패를 숨기는 작업이 아니라 클라이언트에는 일관된 계약을, 운영자에게는 진단 가능한 기록을 제공하는 설계입니다.

**핵심 키워드:** error middleware, next(error), 404, status code, observability

## 핵심 내용

- Express 오류 처리 미들웨어는 `err, req, res, next` 네 인자를 받습니다.
- `next(error)`는 일반 미들웨어를 건너뛰고 오류 처리 흐름으로 전달합니다.
- 존재하지 않는 라우트의 404는 라우트 탐색이 끝난 뒤 별도로 처리합니다.
- 운영 응답에는 스택, 내부 경로와 비밀정보를 노출하지 않습니다.
- 로그에는 요청 식별자와 원인을 남기되 민감한 입력은 마스킹합니다.

## 오류 전달

```javascript
app.get("/reports/:reportId", async (req, res, next) => {
  try {
    const report = await findReport(req.params.reportId);
    if (!report) {
      return res.status(404).json({ error: "report_not_found" });
    }
    return res.json(report);
  } catch (error) {
    return next(error);
  }
});
```

- **목적:** 예상 가능한 자원 없음과 예상하지 못한 시스템 실패를 구분합니다.
- **흐름:** 조회 → 결과 없음은 404 → 성공 응답 → 예외는 오류 미들웨어입니다.
- **결과:** 클라이언트가 실패 종류에 맞게 대응할 수 있습니다.
- **실무 포인트:** 사용 중인 Express 버전과 비동기 오류 전달 방식을 확인해 Promise 거부가 누락되지 않게 합니다.

## 404와 오류 미들웨어 순서

```javascript
app.use((req, res) => {
  res.status(404).json({ error: "route_not_found" });
});

app.use((err, req, res, next) => {
  console.error({ requestId: req.requestId, err });
  res.status(500).json({ error: "internal_server_error" });
});
```

| 처리 | 의미 | 일반 위치 |
|---|---|---|
| 라우트 내부 404 | 경로는 맞지만 자원이 없음 | 해당 컨트롤러 |
| 전역 404 | 어떤 라우트도 경로를 처리하지 않음 | 모든 라우트 뒤 |
| 오류 미들웨어 | 전달된 오류를 최종 응답으로 변환 | 404 처리 뒤 또는 정책에 맞는 마지막 영역 |

오류 미들웨어가 이미 응답을 보낸 상태를 만날 수 있는 구조라면 `res.headersSent`를 확인하고 기본 처리기로 위임하는 전략도 고려합니다.

## 오류 응답 계약

클라이언트에 노출할 코드와 메시지를 분리하면 내부 오류가 바뀌어도 안정적인 처리가 가능합니다.

```json
{
  "error": "validation_failed",
  "message": "입력값을 확인해 주세요",
  "requestId": "..."
}
```

## 실습

1. 일반 오류 미들웨어의 네 인자를 올바른 순서로 작성하세요.
2. 존재하지 않는 API 경로에 JSON 404를 반환하세요.
3. 운영 오류 응답에서 제외할 정보를 세 가지 적으세요.

<details>
<summary>답</summary>

```javascript
app.use((req, res) => res.status(404).json({ error: "route_not_found" }));
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "internal_server_error" });
});
```

스택 추적, 로컬 파일 경로, 토큰·개인정보와 데이터베이스 상세 오류를 응답에 노출하지 않습니다.

</details>

## 더 알아보기

- [미들웨어 파이프라인](04-middleware-pipeline.md)
- [MVC 기반 CRUD와 API 테스트](07-mvc-crud-and-api-testing.md)

## 체크리스트

- [ ] 예상 오류와 시스템 오류를 구분한다.
- [ ] 비동기 오류를 오류 흐름으로 전달한다.
- [ ] 라우트 뒤에 전역 404를 둔다.
- [ ] 오류 미들웨어의 네 인자를 유지한다.
- [ ] 민감한 내부 정보를 응답에서 제거한다.

## 복습 질문 및 답변

### Q1. `next()`와 `next(error)`는 무엇이 다른가요?

<details>
<summary>답</summary>

`next()`는 다음 일반 처리기로 진행하고, `next(error)`는 오류 처리 미들웨어를 찾는 흐름으로 전환합니다.

</details>

### Q2. 라우트가 없을 때 자동으로 오류 미들웨어가 404를 만들까요?

<details>
<summary>답</summary>

일반적으로 라우트가 매칭되지 않은 것은 전달된 오류가 아니므로 모든 라우트 뒤에 404 응답 미들웨어를 명시합니다.

</details>

### Q3. 클라이언트 응답과 서버 로그가 달라야 하는 이유는 무엇인가요?

<details>
<summary>답</summary>

클라이언트에는 안전하고 안정적인 계약이 필요하고, 서버 운영에는 원인을 진단할 상세 문맥이 필요하기 때문입니다.

</details>

## 요약

Express 오류 설계는 라우트의 예상 실패, 전역 404와 시스템 오류를 구분합니다. 안전한 응답 계약과 진단 가능한 로그를 함께 설계해야 합니다.
