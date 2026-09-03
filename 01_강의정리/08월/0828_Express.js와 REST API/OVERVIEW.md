# Express.js와 REST API

웹과 HTTP의 요청·응답부터 Express 라우팅과 미들웨어, 오류 처리, REST API 설계와 MVC 기반 CRUD 테스트까지 백엔드의 기본 흐름을 학습합니다.

## 학습 목표

- HTTP 요청·응답과 CSR·SSR의 차이를 설명합니다.
- Express app과 Router로 경로를 구조화합니다.
- params, query, body와 headers를 검증해 사용합니다.
- 일반·오류 미들웨어의 순서와 흐름을 이해합니다.
- REST·JSON 계약을 MVC 구조와 API 테스트로 구현합니다.

## 추천 학습 순서

1. [웹과 HTTP 기초](01-web-and-http-foundations.md)
2. [Express 애플리케이션과 Router](02-express-app-and-router.md)
3. [요청 데이터와 응답 작성](03-request-data-and-responses.md)
4. [미들웨어 파이프라인](04-middleware-pipeline.md)
5. [오류 처리와 404](05-error-handling-and-not-found.md)
6. [REST API와 JSON 설계](06-rest-api-and-json-design.md)
7. [MVC 기반 CRUD와 API 테스트](07-mvc-crud-and-api-testing.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
HTTP 요청 → 공통 미들웨어 → Router → 입력 검증
→ Controller → Service·Model → 상태 코드와 JSON 응답
→ API 계약 테스트와 오류 관측
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| HTTP, CSR, SSR | 01 웹과 HTTP 기초 |
| app, Router, route handler | 02 Express 애플리케이션과 Router |
| params, query, body, response | 03 요청 데이터와 응답 작성 |
| next, 적용 범위, 팩터리 | 04 미들웨어 파이프라인 |
| next(error), 404, 안전한 500 | 05 오류 처리와 404 |
| 메서드, 자원 URL, JSON | 06 REST API와 JSON 설계 |
| Controller, CRUD, API 테스트 | 07 MVC 기반 CRUD와 API 테스트 |

## 최종 점검

- [ ] URL과 HTTP 메서드로 자원과 행위를 구분한다.
- [ ] 입력을 신뢰하지 않고 타입·범위·권한을 검증한다.
- [ ] 미들웨어 순서와 오류 전달을 확인한다.
- [ ] 일관된 상태 코드와 JSON 오류를 반환한다.
- [ ] 성공과 실패 API 계약을 자동 테스트한다.
