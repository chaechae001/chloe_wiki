# 비동기 처리, 페이지네이션과 프로세스 운영

데이터베이스 작업은 실패할 수 있으므로 비동기 오류를 중앙에서 처리하고, 목록은 제한된 범위만 조회하며, 운영 프로세스는 장애 후 복구 가능하게 구성해야 합니다.

**핵심 키워드:** async middleware, pagination, query, PM2, graceful shutdown

## 비동기 라우터 래퍼

```javascript
const asyncHandler = (handler) => (req, res, next) => {
  Promise.resolve(handler(req, res, next)).catch(next);
};

app.use((error, req, res, next) => {
  const status = error.status ?? 500;
  res.status(status).render("error", {
    message: status === 500 ? "처리 중 오류가 발생했습니다." : error.message,
  });
});
```

운영 응답에는 내부 스택이나 데이터베이스 오류 원문을 노출하지 않고, 서버 로그에는 요청 식별자와 원인을 구조화해 남깁니다.

## 페이지네이션

```javascript
const page = Math.max(1, Number.parseInt(req.query.page, 10) || 1);
const perPage = Math.min(50, Math.max(1, Number.parseInt(req.query.perPage, 10) || 10));
const filter = { published: true };

const [items, total] = await Promise.all([
  Article.find(filter).sort({ createdAt: -1 }).skip((page - 1) * perPage).limit(perPage),
  Article.countDocuments(filter),
]);
```

목록과 개수 쿼리에는 같은 필터를 사용합니다. 데이터가 매우 커지면 깊은 `skip` 대신 정렬 키 기반 커서 페이지네이션을 고려합니다.

## PM2와 종료 흐름

프로세스 관리자는 예기치 않은 종료 뒤 재시작, 로그 관리와 다중 인스턴스 운영을 돕습니다. 그러나 오류를 숨기는 도구가 아닙니다. 종료 신호에서는 새 요청을 막고 진행 요청을 정리한 뒤 DB 연결을 닫습니다.

## 실습

1. `page=-2`, `perPage=1000` 입력을 안전하게 보정하세요.
2. 500 응답과 서버 로그에 각각 무엇을 남길지 정하세요.
3. 종료 순서를 작성하세요.

<details>
<summary>답</summary>

페이지는 최소 1, 크기는 1~50처럼 제한합니다. 사용자에게는 일반화된 메시지를, 로그에는 요청 ID·오류 종류·스택을 남깁니다. 종료는 요청 수신 중단 → 진행 요청 대기 → DB 종료 순서입니다.

</details>

## 체크리스트

- [ ] 비동기 오류를 중앙 처리기로 전달한다.
- [ ] 내부 오류 정보를 응답에 노출하지 않는다.
- [ ] 페이지와 크기에 상한·하한을 둔다.
- [ ] 목록과 개수에 같은 필터를 쓴다.
- [ ] 재시작과 안전한 종료 정책을 함께 둔다.

## 복습 질문 및 답변

### Q1. `try/catch`를 모든 라우터에 반복하지 않는 방법은 무엇인가요?

<details>
<summary>답</summary>

Promise 거부를 `next`로 전달하는 공통 래퍼를 적용하고 마지막에 오류 미들웨어를 둡니다.

</details>

### Q2. 페이지 크기를 제한해야 하는 이유는 무엇인가요?

<details>
<summary>답</summary>

과도한 조회가 메모리와 데이터베이스 부하를 일으켜 다른 요청까지 지연시키는 것을 막기 위해서입니다.

</details>

### Q3. 프로세스 자동 재시작만 있으면 장애 대응이 충분한가요?

<details>
<summary>답</summary>

아닙니다. 반복 장애를 탐지할 로그·지표·알림과 원인 수정, 준비 상태 점검이 함께 필요합니다.

</details>

## 요약

서비스 안정성은 오류 전달, 제한된 조회, 관측 가능한 프로세스 수명주기를 한 흐름으로 묶을 때 높아집니다.
