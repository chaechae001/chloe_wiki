# Express.js와 MongoDB 웹서비스 구현

Pug 기반 게시판에서 시작해 MongoDB CRUD, 페이지네이션, 회원가입·세션 인증, 작성자 권한과 댓글 API, JWT·계정 복구·OAuth, 배포 경계까지 하나의 웹서비스가 확장되는 흐름을 학습합니다.

## 학습 목표

- Express와 Pug로 안전한 서버 렌더링 화면을 구성합니다.
- Mongoose 게시판 CRUD에 검증·권한·오류 처리를 적용합니다.
- 페이지네이션과 프로세스 수명주기를 운영 관점에서 설계합니다.
- 비밀번호 해시와 Passport 세션 인증을 구현 원리로 이해합니다.
- 댓글 API, JWT, 계정 복구, OAuth와 프록시의 보안 경계를 설명합니다.

## 추천 학습 순서

1. [Pug로 서버 렌더링 구성하기](01-pug-server-rendering.md)
2. [게시판 CRUD 아키텍처](02-board-crud-architecture.md)
3. [비동기 처리, 페이지네이션과 프로세스 운영](03-async-pagination-operations.md)
4. [회원가입과 비밀번호 보안](04-signup-password-security.md)
5. [Passport와 세션 인증](05-passport-session-authentication.md)
6. [작성자 권한, 댓글 API와 Aggregation](06-authorization-comments-aggregation.md)
7. [JWT, 계정 복구, OAuth와 배포 경계](07-token-recovery-oauth-deployment.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
요청 → 라우팅 → 입력 검증 → 인증·인가 → 서비스 규칙
→ Mongoose 조회·변경 → Pug HTML 또는 JSON 응답
→ 오류 관측 → 프록시·프로세스·DB 수명주기 관리
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| Pug, layout, mixin, SSR | 01 서버 렌더링 |
| 게시글 생성·조회·수정·삭제 | 02 CRUD 아키텍처 |
| 오류 래퍼, 목록 분할, PM2 | 03 비동기·운영 |
| 가입 검증, 비밀번호 해시 | 04 회원가입 보안 |
| LocalStrategy, 세션, 쿠키 | 05 Passport 인증 |
| 작성자 권한, 댓글, 집계 | 06 관계 기능 |
| JWT, 재설정, OAuth, Nginx | 07 인증 확장·배포 |

## 최종 점검

- [ ] 사용자 입력과 식별자를 서버에서 검증한다.
- [ ] 인증과 소유권 기반 인가를 모두 적용한다.
- [ ] 비밀번호·토큰·연결 문자열을 로그와 응답에서 제외한다.
- [ ] 쿠키, 세션, JWT의 만료와 폐기 정책을 정한다.
- [ ] 오류·프로세스·프록시·DB 종료 흐름을 관측한다.
