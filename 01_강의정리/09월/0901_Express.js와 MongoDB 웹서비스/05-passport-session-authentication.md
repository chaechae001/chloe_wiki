# Passport와 세션 인증

세션 로그인은 자격 증명을 한 번 확인한 뒤 브라우저에는 예측 불가능한 세션 식별자만 전달하고, 서버 측 저장소에서 로그인 상태를 관리합니다.

**핵심 키워드:** Passport, LocalStrategy, session, cookie, serialize

## 인증 흐름

```text
로그인 폼 → LocalStrategy에서 사용자·비밀번호 확인
→ 세션에 최소 식별자 저장 → 다음 요청의 쿠키 확인
→ 사용자 복원 → req.user 사용
```

```javascript
passport.serializeUser((user, done) => done(null, user.id));

passport.deserializeUser(async (id, done) => {
  try {
    done(null, await User.findById(id).select("name role"));
  } catch (error) {
    done(error);
  }
});
```

세션에는 사용자 문서 전체가 아니라 식별자처럼 작은 값을 저장합니다. 세션 미들웨어 뒤에 `passport.initialize()`와 `passport.session()`을 배치합니다.

## 쿠키와 세션 저장소

운영 환경에서는 `httpOnly`, `secure`, 적절한 `sameSite`, 만료 시간을 설정합니다. 기본 메모리 저장소는 다중 프로세스와 재시작에 적합하지 않으므로 공유 가능한 운영용 저장소를 사용합니다. 로그인 성공 후에는 세션 고정 공격을 줄이도록 세션 ID를 재생성합니다.

## 접근 제어 미들웨어

```javascript
function requireLogin(req, res, next) {
  if (req.isAuthenticated?.()) return next();
  res.redirect("/login");
}
```

인증은 “누구인가”, 인가는 “이 작업을 해도 되는가”를 판단합니다. 로그인 확인만으로 다른 사용자의 글 수정이 허용되어서는 안 됩니다.

## 실습

1. 미들웨어 등록 순서를 작성하세요.
2. 운영 쿠키 옵션과 HTTPS 관계를 설명하세요.
3. 로그아웃 때 제거해야 할 상태를 적으세요.

<details>
<summary>답</summary>

세션 → Passport 초기화 → Passport 세션 복원 → 라우터 순서로 둡니다. `secure` 쿠키는 HTTPS에서만 전송되며, 로그아웃 시 Passport 상태와 서버 세션을 제거하고 쿠키도 만료시킵니다.

</details>

## 체크리스트

- [ ] 로그인 실패를 일관되게 처리한다.
- [ ] 세션에는 최소 식별자만 저장한다.
- [ ] 운영용 공유 세션 저장소를 사용한다.
- [ ] 쿠키 보안 옵션을 설정한다.
- [ ] 인증과 인가를 분리해 검사한다.

## 복습 질문 및 답변

### Q1. `serializeUser`의 역할은 무엇인가요?

<details>
<summary>답</summary>

로그인한 사용자에서 세션에 저장할 최소 값을 선택합니다.

</details>

### Q2. `deserializeUser`는 언제 실행되나요?

<details>
<summary>답</summary>

후속 요청에서 세션 식별자를 바탕으로 현재 사용자 정보를 복원할 때 실행됩니다.

</details>

### Q3. 메모리 세션 저장소가 운영에 부적합한 이유는 무엇인가요?

<details>
<summary>답</summary>

재시작 시 상태가 사라지고 여러 인스턴스가 같은 세션을 공유하기 어려우며 메모리 관리 문제도 생길 수 있습니다.

</details>

## 요약

세션 인증은 미들웨어 순서, 최소 직렬화, 안전한 쿠키와 공유 저장소를 함께 설계해야 신뢰할 수 있습니다.
