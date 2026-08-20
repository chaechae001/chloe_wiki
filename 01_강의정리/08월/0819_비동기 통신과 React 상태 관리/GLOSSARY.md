# 용어집

이번 학습에서 등장한 비동기 통신과 상태 관리 용어를 쉬운 말로 정리했습니다.

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| Call Stack | 현재 실행 중인 JavaScript 함수가 쌓이는 실행 공간 | [이벤트 루프와 Promise](01-event-loop-and-promises.md) |
| Event Loop | Stack이 비었을 때 대기 작업이 실행되도록 연결하는 흐름 | [이벤트 루프와 Promise](01-event-loop-and-promises.md) |
| Promise | 미래에 성공 값이나 실패 이유가 결정되는 비동기 결과 객체 | [이벤트 루프와 Promise](01-event-loop-and-promises.md) |
| Settled | Promise가 fulfilled 또는 rejected로 확정된 상태 | [async/await와 Promise 조합](02-async-await-and-promise-combinators.md) |
| Endpoint | API가 특정 자원이나 기능을 제공하는 요청 주소 | [HTTP API와 CORS](03-http-api-openapi-and-cors.md) |
| OpenAPI | API 요청과 응답 계약을 구조적으로 기록하는 표준 | [HTTP API와 CORS](03-http-api-openapi-and-cors.md) |
| Origin | URL의 프로토콜, 호스트, 포트 조합 | [HTTP API와 CORS](03-http-api-openapi-and-cors.md) |
| CORS | 브라우저의 교차 출처 요청을 서버 허용 정책으로 제어하는 방식 | [HTTP API와 CORS](03-http-api-openapi-and-cors.md) |
| Prop Drilling | 사용하지 않는 중간 컴포넌트를 거쳐 props를 깊게 전달하는 현상 | [React 상태 관리와 Context](04-react-state-and-context.md) |
| Context | React 트리 범위에서 공통 값을 직접 읽게 하는 기능 | [React 상태 관리와 Context](04-react-state-and-context.md) |
| Flux | Action부터 View까지 단방향 상태 흐름을 강조하는 아키텍처 패턴 | [Flux와 useReducer](05-flux-and-usereducer.md) |
| Action | 상태 시스템에서 발생한 사건을 설명하는 값 | [Flux와 useReducer](05-flux-and-usereducer.md) |
| Reducer | 현재 상태와 Action으로 다음 상태를 계산하는 순수 함수 | [Flux와 useReducer](05-flux-and-usereducer.md) |
| Store | Redux가 앱의 공유 상태 트리를 보관하는 곳 | [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md) |
| Selector | Store에서 필요한 상태나 계산 결과를 읽는 함수 | [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md) |
| Middleware | Action이 Reducer에 도달하기 전 로깅·비동기 처리 등을 수행하는 확장 지점 | [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md) |
| Thunk | 함수를 dispatch해 비동기 로직을 실행하도록 하는 패턴 | [Redux 비동기 상태 관리](07-redux-async-state.md) |
| Race condition | 여러 비동기 결과의 도착 순서 때문에 최신 상태가 잘못 덮이는 문제 | [Redux 비동기 상태 관리](07-redux-async-state.md) |
