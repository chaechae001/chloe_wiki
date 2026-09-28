# FastAPI StreamingResponse

> `StreamingResponse`에 generator를 넘기면 서버가 generator의 조각을 HTTP 본문으로 순서대로 전송합니다.

`StreamingResponse` · `media_type` · `httpx.stream` · `SSE` · `backpressure`

## 핵심요약

- endpoint는 generator 또는 async generator를 `StreamingResponse`로 감쌉니다.
- `text/plain`은 단순 텍스트 조각 전달에 적합합니다.
- SSE는 `text/event-stream`과 `data: ...\n\n` 프레임을 사용합니다.
- 클라이언트도 stream API로 읽어야 조각이 즉시 보입니다.
- proxy buffering과 연결 timeout은 배포 환경에서 별도 점검합니다.

## 1. 더미 스트림부터 검증

```python
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.get("/demo/stream")
async def demo_stream():
    async def generate():
        for index in range(3):
            await asyncio.sleep(0.2)
            yield f"chunk {index}\n"
    return StreamingResponse(generate(), media_type="text/plain")
```

먼저 더미 generator로 네트워크 전달 자체를 확인한 뒤 LLM source를 넣으면 오류 원인을 더 쉽게 분리할 수 있습니다.

<details>
<summary>답</summary>

응답 body를 미리 모두 만들지 않고 generator가 yield하는 시점마다 전송합니다.

</details>

## 2. 클라이언트도 스트림으로 읽기

```python
import httpx

with httpx.Client(timeout=20.0) as client:
    with client.stream("GET", "http://127.0.0.1:8000/demo/stream") as response:
        response.raise_for_status()
        for text in response.iter_text():
            print(text, end="")
```

일반 `get()`으로 body를 전부 받은 뒤 출력하면 서버가 스트림이어도 화면의 실시간 효과를 잃습니다.

## 3. SSE 선택

SSE는 서버에서 브라우저로 이벤트를 보내는 단방향 표준입니다. GET 기반 `EventSource`는 간편하지만 header나 POST body가 필요한 chat 요청에는 fetch reader 또는 SSE 클라이언트 라이브러리를 고려합니다.

## 직접 해보기

1. 0.2초 간격의 더미 endpoint를 만드세요.
2. `iter_text()`로 시간 차이를 출력하세요.
3. 같은 값을 SSE `data:` 프레임으로 바꾸세요.

<details>
<summary>정답 보기</summary>

`media_type="text/event-stream"`과 `yield f"data: {piece}\n\n"`를 사용합니다. 클라이언트는 SSE framing을 해석해야 합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `text/plain` vs SSE | 단순 body 조각과 이벤트 규약이 있는 전송입니다. |
| server stream vs client stream | 보내는 방식과 도착 즉시 읽는 방식입니다. |
| local 동작 vs proxy 배포 | 개발 서버와 buffering·timeout이 있는 운영 경로입니다. |

## 연결되는 개념

- 이전: [async generator 파이프라인](03-async-generator-pipelines.md)
- 다음: [chat stream endpoint 설계](05-chat-stream-endpoint.md)

## 셀프 체크

- [ ] generator를 StreamingResponse로 감싼다.
- [ ] media type을 목적에 맞게 선택한다.
- [ ] httpx stream 읽기를 사용한다.
- [ ] SSE frame을 설명한다.
- [ ] proxy buffering을 점검한다.

### 복습 질문 및 답변

**Q1. StreamingResponse만 쓰면 브라우저도 자동으로 실시간 표시하나요?**

<details>
<summary>답</summary>

클라이언트가 response body를 stream으로 읽고 UI를 갱신해야 합니다.

</details>

**Q2. SSE는 WebSocket인가요?**

<details>
<summary>답</summary>

아닙니다. SSE는 서버에서 클라이언트로의 단방향 HTTP 이벤트 흐름입니다.

</details>

**Q3. 운영에서 한 번에 도착하면 어디를 확인하나요?**

<details>
<summary>답</summary>

클라이언트 읽기 방식, 응답 header, CDN·reverse proxy의 buffering 설정과 timeout을 확인합니다.

</details>

## 한 줄 정리

> StreamingResponse는 generator의 흐름을 HTTP에 연결하고, 클라이언트의 스트림 소비가 이를 완성합니다.
