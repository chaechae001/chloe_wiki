# 용어집

| 용어 | 설명 |
|---|---|
| 스트리밍 | 생성 중인 데이터를 조각 단위로 전달하는 방식 |
| TTFT | 요청부터 첫 토큰이 표시될 때까지의 시간 |
| token | 모델이 텍스트를 처리·생성하는 단위 |
| chunk | 네트워크로 전달되는 데이터 묶음 |
| SSE | 서버가 클라이언트에 이벤트를 보내는 단방향 HTTP 흐름 |
| Responses API | OpenAI의 통합 응답 API |
| delta | 이번 stream event에서 추가된 텍스트 조각 |
| async generator | `async def`와 `yield`로 만드는 비동기 조각 생산자 |
| StreamingResponse | generator를 HTTP 응답 body로 전송하는 FastAPI 응답 |
| cancellation | 소비자가 스트림을 중단한 상태 |
| `CancelledError` | 비동기 task 취소를 알리는 예외 |
| timeout | 대기 가능한 최대 시간 |
| buffering | 조각을 즉시 보내지 않고 모아 두는 동작 |
| sentinel | 완료·오류 등을 나타내기 위한 약속된 표시 값 |
| contract test | status, schema, event 순서 등 API 약속을 검증하는 test |

## 관련 문서

- [전체 개요](OVERVIEW.md)
- [스트리밍 응답의 원리](01-streaming-fundamentals.md)
- [스트리밍 오류·취소·결과 점검](06-stream-errors-cancellation-testing.md)
