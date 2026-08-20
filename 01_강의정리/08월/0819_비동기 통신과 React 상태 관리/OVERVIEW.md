# 비동기 통신과 React 상태 관리

> 서버의 데이터를 기다리는 과정부터 앱 전체 상태가 바뀌는 경로까지 하나의 흐름으로 연결합니다.

비동기 통신은 데이터를 가져오는 기술이고 상태 관리는 그 결과를 UI에 예측 가능하게 반영하는 기술입니다. Promise의 실행 모델에서 시작해 API 계약, React 상태 범위, Redux의 데이터 흐름 순서로 학습합니다.

## 학습 로드맵

```mermaid
flowchart LR
    A["Event Loop"] --> B["Promise와 async/await"]
    B --> C["HTTP API와 CORS"]
    C --> D["React 상태와 Context"]
    D --> E["Flux와 Reducer"]
    E --> F["Redux Toolkit"]
    F --> G["Redux 비동기 상태"]
```

## 목차

| # | 글 | 한 줄 소개 | 활용도 |
|---|---|---|---|
| 1 | [이벤트 루프와 Promise](01-event-loop-and-promises.md) | 비동기 실행 순서와 Promise 상태 이해 | ★★★★★ |
| 2 | [async/await와 Promise 조합](02-async-await-and-promise-combinators.md) | 순차·동시 요청과 실패 정책 설계 | ★★★★★ |
| 3 | [HTTP API와 CORS](03-http-api-openapi-and-cors.md) | 요청 계약과 교차 출처 정책 이해 | ★★★★★ |
| 4 | [React 상태 관리와 Context](04-react-state-and-context.md) | 상태의 위치와 공유 범위 결정 | ★★★★★ |
| 5 | [Flux와 useReducer](05-flux-and-usereducer.md) | Action 기반 상태 변경 구조화 | ★★★★☆ |
| 6 | [Redux와 Redux Toolkit](06-redux-and-redux-toolkit.md) | Store와 slice 기반 전역 상태 구성 | ★★★★★ |
| 7 | [Redux 비동기 상태 관리](07-redux-async-state.md) | 요청 생명주기를 Store와 UI에 연결 | ★★★★★ |

## 다루는 핵심 개념

- Event Loop와 Promise의 성공·실패 흐름
- async/await과 여러 Promise의 완료 정책
- HTTP 요청, OpenAPI 계약, CORS 정책
- useState, useRef, Context의 선택 기준
- Flux, useReducer, Redux의 단방향 데이터 흐름
- Redux Toolkit과 createAsyncThunk의 비동기 상태

## 학습 포인트

- 실행 순서와 요청 완료 순서를 구분합니다.
- 상태 값뿐 아니라 loading과 error도 함께 모델링합니다.
- 로컬 상태, Context, Redux를 복잡도에 맞게 선택합니다.
- Reducer는 순수하게 유지하고 부수 효과는 바깥에서 실행합니다.
- 공개 API 계약과 화면 상태의 경계를 분리해 설계합니다.

## 함께 보면 좋은 자료

- [용어집](GLOSSARY.md)
