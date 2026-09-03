# GLOSSARY

## 용어 정리

| 용어 | 설명 |
|---|---|
| Web | 네트워크에서 URL과 HTTP를 통해 자원과 문서를 연결하는 시스템 |
| HTTP | 클라이언트와 서버가 요청·응답을 교환하는 애플리케이션 프로토콜 |
| Request | 클라이언트가 메서드, 경로, 헤더와 본문으로 전달하는 메시지 |
| Response | 서버가 상태, 헤더와 본문으로 반환하는 메시지 |
| CSR | 브라우저가 JavaScript로 화면을 주로 구성하는 렌더링 방식 |
| SSR | 서버가 요청에 맞는 HTML을 주로 생성하는 렌더링 방식 |
| Express.js | Node.js에서 라우팅과 미들웨어를 제공하는 웹 프레임워크 |
| Application Object | Express 설정, 미들웨어와 라우트를 연결하는 app 객체 |
| Router | 관련 라우트와 미들웨어를 묶어 마운트하는 객체 |
| Route | HTTP 메서드와 경로를 처리 함수에 연결한 규칙 |
| Route Handler | 최종 응답을 만들거나 오류를 전달하는 요청 처리 함수 |
| Path Parameter | URL 경로 안에서 특정 자원을 식별하는 값 |
| Query Parameter | 필터, 정렬과 페이지 같은 선택 조건을 나타내는 값 |
| Request Body | 생성·수정할 구조화 데이터를 담는 요청 본문 |
| Middleware | 요청과 응답 사이에서 공통 작업을 수행하는 함수 |
| next | 다음 미들웨어로 진행하거나 오류 흐름으로 전환하는 함수 |
| Error Middleware | 네 인자로 오류를 받아 응답과 로깅을 처리하는 미들웨어 |
| REST | 자원 중심 인터페이스와 HTTP 의미를 활용하는 아키텍처 스타일 |
| Resource | API가 식별하고 표현하는 도메인 대상 |
| JSON | 객체와 배열을 표현하는 텍스트 데이터 교환 형식 |
| CRUD | Create, Read, Update, Delete의 데이터 기본 작업 |
| MVC | Model, View, Controller의 책임을 분리하는 설계 패턴 |
| Controller | HTTP 입력을 처리하고 결과를 응답으로 변환하는 계층 |
| Model | 데이터 구조, 상태 또는 저장소 접근을 담당하는 계층 |
| Status Code | HTTP 처리 결과의 종류를 나타내는 숫자 코드 |

## 연결해서 기억하기

클라이언트가 HTTP 요청을 보내면 Express 앱의 미들웨어와 Router가 순서대로 실행됩니다. Controller는 검증된 입력으로 비즈니스와 데이터 계층을 호출하고, REST 계약에 맞는 상태 코드와 JSON 응답을 돌려줍니다.

## 관련 학습

- [웹과 HTTP 기초](01-web-and-http-foundations.md)
- [미들웨어 파이프라인](04-middleware-pipeline.md)
- [REST API와 JSON 설계](06-rest-api-and-json-design.md)
- [MVC 기반 CRUD와 API 테스트](07-mvc-crud-and-api-testing.md)
