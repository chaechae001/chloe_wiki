# 용어집

Ajax와 REST API 학습에서 반복해서 등장하는 용어를 쉬운 말로 정리했습니다.

## 웹 통신

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 클라이언트 | 서버의 기능이나 데이터를 요청해 사용하는 프로그램입니다. 브라우저가 대표적입니다. | [클라이언트·서버와 HTTP 통신](01-client-server-and-http.md) | 프론트엔드, 요청 |
| 서버 | 요청을 해석하고 로직이나 데이터 처리를 수행해 응답하는 프로그램입니다. | [클라이언트·서버와 HTTP 통신](01-client-server-and-http.md) | 백엔드, 응답 |
| 프로토콜 | 통신 참여자가 메시지의 형식과 순서를 이해하도록 정한 공통 규칙입니다. | [클라이언트·서버와 HTTP 통신](01-client-server-and-http.md) | HTTP |
| HTTP | 웹에서 요청과 응답을 교환하기 위한 대표적인 응용 계층 프로토콜입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | 메서드, 상태 코드 |
| 무상태 | 서버가 이전 요청의 문맥을 기본적으로 기억하지 않고 각 요청을 독립적으로 처리하는 성질입니다. | [클라이언트·서버와 HTTP 통신](01-client-server-and-http.md) | 토큰, 쿠키 |

## HTTP 메시지

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| HTTP 메서드 | 조회·생성·수정·삭제처럼 자원에 원하는 행위를 나타내는 값입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | GET, POST, PATCH |
| 헤더 | 본문 형식, 인증, 캐시 등 메시지 처리에 필요한 부가 정보입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | Content-Type |
| 본문 | 요청에서 보낼 데이터나 응답에서 받은 실제 데이터가 들어가는 영역입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | JSON |
| 상태 코드 | 서버가 요청을 어떻게 처리했는지 세 자리 숫자로 요약한 값입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | 2xx, 4xx, 5xx |
| JSON | 객체와 배열 형태의 데이터를 문자열로 교환할 때 널리 쓰는 텍스트 형식입니다. | [HTTP 요청과 응답 메시지](02-http-messages.md) | 직렬화, 역직렬화 |

## API와 비동기 요청

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| API | 프로그램이 다른 프로그램의 기능을 정해진 방식으로 사용할 수 있게 만든 접점입니다. | [자원 중심의 REST API 설계](03-rest-api-design.md) | Web API |
| REST | HTTP를 활용해 자원을 일관된 인터페이스로 다루는 아키텍처 스타일입니다. | [자원 중심의 REST API 설계](03-rest-api-design.md) | 자원, 표현 |
| URI | API가 다루는 자원을 고유하게 식별하는 문자열입니다. | [자원 중심의 REST API 설계](03-rest-api-design.md) | URL, 경로 |
| Ajax | 전체 페이지를 새로고침하지 않고 서버와 데이터를 주고받아 필요한 화면만 갱신하는 방식입니다. | [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md) | 비동기 통신 |
| XMLHttpRequest | 상태 변화와 이벤트를 직접 다뤄 Ajax 요청을 보내는 전통적인 브라우저 API입니다. | [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md) | XHR |
| Fetch API | Promise 기반으로 HTTP 요청과 Response를 다루는 브라우저 기본 API입니다. | [Ajax와 Fetch API](04-ajax-xhr-and-fetch.md) | response.ok |
| Axios | 요청 설정과 응답 객체 처리를 제공하는 별도 HTTP 클라이언트 라이브러리입니다. | [Axios 요청 패턴](05-axios-request-patterns.md) | data, 인터셉터 |
| 응답 객체 | 상태, 헤더, 본문 등 서버 응답 정보를 묶은 객체입니다. Axios에서는 본문이 주로 data에 있습니다. | [Axios 요청 패턴](05-axios-request-patterns.md) | 구조 분해 할당 |

## 화면 반영

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `preventDefault()` | 폼 제출처럼 브라우저가 원래 수행하려는 이벤트 동작을 막는 메서드입니다. | [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md) | submit 이벤트 |
| `find()` | 조건을 만족하는 첫 번째 배열 요소 하나를 반환하는 메서드입니다. | [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md) | undefined |
| `filter()` | 조건을 만족하는 모든 배열 요소를 새 배열로 반환하는 메서드입니다. | [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md) | 목록 가공 |
| 문서 조각 | 여러 DOM 노드를 메모리에서 모은 뒤 화면에 한 번에 붙이기 위한 임시 컨테이너입니다. | [API 데이터와 DOM 렌더링](06-api-data-and-dom-rendering.md) | DocumentFragment |

## 빠른 비교

| 비교 | 핵심 차이 |
|---|---|
| Ajax vs Fetch | Ajax는 통신 방식이고 Fetch는 그 방식을 구현하는 API입니다. |
| Fetch vs Axios | Fetch는 브라우저 기본 API, Axios는 별도 라이브러리입니다. |
| URI vs URL | URI는 식별자, URL은 위치와 접근 방법까지 포함하는 URI입니다. |
| `find` vs `filter` | find는 하나, filter는 여러 항목의 배열을 반환합니다. |
