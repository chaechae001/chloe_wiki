# 미들웨어 파이프라인

Express 미들웨어는 요청과 최종 응답 사이에서 공통 관심사를 순서대로 처리하는 함수입니다.

**핵심 키워드:** middleware, next, 순서, factory, router stack

## 핵심 내용

- 일반 미들웨어는 `req`, `res`, `next`를 받습니다.
- 응답을 완료하지 않았다면 `next()`를 호출해야 다음 단계로 이동합니다.
- 등록 순서가 실행 순서이므로 body parser, 인증과 라우트의 위치가 중요합니다.
- 애플리케이션 전체, 특정 경로, Router 또는 개별 라우트에 적용할 수 있습니다.
- 함수형 미들웨어는 설정을 받아 서로 다른 동작의 미들웨어를 생성합니다.

## 기본 흐름

```javascript
function requestTimer(req, res, next) {
  req.startedAt = Date.now();
  next();
}

app.use(requestTimer);
```

- **목적:** 요청 시작 시각을 이후 처리기가 사용할 수 있게 저장합니다.
- **흐름:** 요청 수신 → 값 추가 → next 호출 → 다음 미들웨어 또는 핸들러입니다.
- **결과:** 같은 요청의 뒤 단계에서 `req.startedAt`을 읽을 수 있습니다.
- **실무 포인트:** `next()` 호출 뒤에도 함수는 계속 실행되므로 필요하면 `return next()`로 흐름을 명확히 합니다.

## 적용 범위

| 등록 방식 | 범위 |
|---|---|
| `app.use(middleware)` | 이후의 모든 요청 |
| `app.use("/admin", middleware)` | 특정 경로 접두사 |
| `router.use(middleware)` | 해당 Router |
| `router.get("/", middleware, handler)` | 특정 라우트 |

## 미들웨어 팩터리

```javascript
function requireRole(role) {
  return function roleMiddleware(req, res, next) {
    if (req.user?.role !== role) {
      return res.status(403).json({ error: "forbidden" });
    }
    return next();
  };
}

app.use("/admin", requireRole("admin"), adminRouter);
```

팩터리는 설정을 클로저에 보관해 같은 구현을 여러 모드로 재사용합니다. 모드별 데이터 접근 권한과 기본값이 섞이지 않도록 분기와 테스트를 명확히 합니다.

## 서브 스택과 라우트 핸들러

한 경로에 여러 함수를 나열하면 작은 파이프라인이 됩니다. 인증, 입력 검증, 컨트롤러처럼 책임별로 나눌 수 있지만 공유 상태를 `req`에 추가할 때 속성 이름 충돌을 피해야 합니다.

## 실습

1. 요청마다 고유한 ID를 추가하는 미들웨어를 작성하세요.
2. 역할을 인자로 받는 미들웨어 팩터리를 작성하세요.
3. `express.json()`, 인증, 라우트의 적절한 등록 순서를 설명하세요.

<details>
<summary>답</summary>

```javascript
const attachRequestId = (req, res, next) => {
  req.requestId = crypto.randomUUID();
  return next();
};
```

JSON 파서가 본문을 만들고 인증이 사용자를 확인한 뒤 권한이 필요한 라우트를 실행하도록 등록합니다. 실제 순서는 각 미들웨어의 입력 의존성에 맞춰 정합니다.

</details>

## 더 알아보기

- [요청 데이터와 응답 작성](03-request-data-and-responses.md)
- [오류 처리와 404](05-error-handling-and-not-found.md)

## 체크리스트

- [ ] next 호출과 응답 종료를 구분한다.
- [ ] 미들웨어 등록 순서를 확인한다.
- [ ] 필요한 범위에만 적용한다.
- [ ] 공통 로직을 팩터리로 재사용한다.
- [ ] req에 추가하는 속성 계약을 문서화한다.

## 복습 질문 및 답변

### Q1. 응답을 보낸 뒤에도 `next()`를 호출해야 하나요?

<details>
<summary>답</summary>

일반적으로 호출하지 않습니다. 응답을 완료했다면 파이프라인을 끝내고, 다음 처리로 넘길 때만 next를 호출합니다.

</details>

### Q2. 미들웨어 순서가 중요한 이유는 무엇인가요?

<details>
<summary>답</summary>

앞 단계가 만든 req 값이나 검증 결과를 뒤 단계가 사용하며, 이미 지나간 미들웨어는 해당 요청에 다시 적용되지 않기 때문입니다.

</details>

### Q3. 미들웨어 팩터리는 무엇을 반환하나요?

<details>
<summary>답</summary>

설정 값을 클로저로 기억하는 실제 Express 미들웨어 함수 `(req, res, next)`를 반환합니다.

</details>

## 요약

미들웨어는 요청 처리를 책임별로 연결하는 파이프라인입니다. 실행 순서, 적용 범위, next와 응답 종료를 명확히 하면 인증·검증·로깅을 재사용할 수 있습니다.
