# 용어집

| 용어 | 설명 |
|---|---|
| SSR | 서버에서 데이터가 결합된 HTML을 만들어 응답하는 렌더링 방식 |
| Template Engine | 데이터와 템플릿을 결합해 문서를 생성하는 도구 |
| Pug | 들여쓰기 기반 Node.js 템플릿 엔진 |
| CRUD | 생성, 조회, 수정, 삭제의 기본 데이터 작업 |
| Middleware | 요청과 응답 사이에서 공통 처리를 수행하는 함수 |
| Pagination | 큰 결과를 일정 단위의 페이지로 나누는 방식 |
| Cursor Pagination | 마지막 정렬 키를 기준으로 다음 범위를 조회하는 방식 |
| Hash | 입력을 고정 길이 값으로 변환하는 단방향 함수 |
| Salt | 같은 비밀번호도 다른 해시가 되게 더하는 무작위 값 |
| Authentication | 사용자가 누구인지 확인하는 인증 과정 |
| Authorization | 인증된 사용자가 작업을 할 수 있는지 판단하는 인가 과정 |
| Session | 서버 측 저장소에서 유지하는 사용자별 상태 |
| Cookie | 브라우저가 요청과 함께 보내는 작은 데이터 |
| Passport | Express 인증 전략을 연결하는 미들웨어 |
| LocalStrategy | 사용자명과 비밀번호로 인증하는 Passport 전략 |
| Populate | Mongoose 참조 필드를 관련 문서로 채우는 기능 |
| Subdocument | 다른 MongoDB 문서 내부에 포함된 중첩 문서 |
| Aggregation Pipeline | 여러 처리 단계를 연결해 MongoDB 데이터를 변환·집계하는 기능 |
| JWT | 서명된 클레임을 담는 토큰 형식 |
| OAuth | 사용자가 외부 서비스의 제한된 권한을 위임하는 표준 |
| CSRF | 사용자의 인증 상태를 악용해 의도하지 않은 요청을 보내게 하는 공격 |
| XSS | 신뢰되지 않은 스크립트가 페이지에서 실행되는 공격 |
| PM2 | Node.js 프로세스 실행과 재시작 등을 관리하는 도구 |
| Reverse Proxy | 클라이언트 요청을 받아 내부 애플리케이션 서버로 전달하는 서버 |
| Nginx | 리버스 프록시, TLS 종료와 정적 파일 제공 등에 쓰이는 웹 서버 |

## 함께 보기

- [Pug로 서버 렌더링 구성하기](01-pug-server-rendering.md)
- [Passport와 세션 인증](05-passport-session-authentication.md)
- [JWT, 계정 복구, OAuth와 배포 경계](07-token-recovery-oauth-deployment.md)
