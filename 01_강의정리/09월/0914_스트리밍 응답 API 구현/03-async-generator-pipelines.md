# async generator 파이프라인

> async generator는 `yield` 사이에서 비동기 I/O를 기다리며 조각을 재방출하는 스트리밍 연결 장치입니다.

`async def` · `yield` · `async for` · `await` · `cancellation`

## 핵심요약

- generator는 값을 한 번에 만들지 않고 `yield`마다 하나씩 냅니다.
- async generator는 `yield` 사이에서 `await`할 수 있습니다.
- 소비는 `for`가 아니라 `async for`입니다.
- 변환·필터·로깅을 pipeline 단계로 나눌 수 있습니다.
- cancellation은 삼키지 말고 정리 후 다시 올립니다.

## 1. 최소 async generator

```python
import asyncio

async def word_stream(text: str):
    for word in text.split():
        await asyncio.sleep(0.1)
        yield word + " "

async for piece in word_stream("조각 단위로 전달합니다"):
    print(piece, end="")
```

`await`가 있는 네트워크·파일·모델 스트림에서 일반 generator보다 자연스럽습니다.

<details>
<summary>답</summary>

호출 즉시 모든 값을 만들지 않고, 소비자가 다음 값을 요청할 때까지 실행을 멈출 수 있습니다.

</details>

## 2. 변환 단계 추가

```python
async def nonempty(source):
    async for piece in source:
        if piece:
            yield piece
```

이 지점에 빈 chunk 가드, 길이 제한, 형식 변환, 운영 metric을 추가할 수 있습니다. 과도한 버퍼링은 스트리밍의 장점을 없애므로 꼭 필요한 상태만 유지합니다.

## 3. 취소 정리

```python
import asyncio

async def guarded(source):
    try:
        async for piece in source:
            yield piece
    except asyncio.CancelledError:
        # 필요한 정리·안전한 로그만 남김
        raise
    finally:
        pass  # resource cleanup
```

## 직접 해보기

1. list를 async generator로 바꾸세요.
2. 빈 문자열을 거르는 변환기를 만드세요.
3. finally에 정리 메시지를 남겨 정상 종료·취소 모두 확인하세요.

<details>
<summary>정답 보기</summary>

원본을 `async for`로 읽어 조건을 통과한 값만 `yield`하고, finally에 연결 정리 코드를 둡니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| generator vs async generator | 동기 순회와 await 가능한 비동기 순회입니다. |
| `for` vs `async for` | 동기 iterator와 async iterator 소비 문법입니다. |
| `return` vs `yield` | 함수 종료 값과 스트림의 다음 조각입니다. |

## 연결되는 개념

- 이전: [OpenAI 스트림 이벤트 처리](02-openai-stream-events.md)
- 다음: [FastAPI StreamingResponse](04-fastapi-streamingresponse.md)

## 셀프 체크

- [ ] async generator 선언을 작성한다.
- [ ] `async for`로 소비한다.
- [ ] yield 사이에 await를 둔다.
- [ ] 빈 조각을 필터링한다.
- [ ] CancelledError를 다시 raise한다.

### 복습 질문 및 답변

**Q1. async generator에서도 `yield`를 쓰나요?**

<details>
<summary>답</summary>

네. `async def` 안에서 `yield`를 사용하고 소비자가 `async for`로 읽습니다.

</details>

**Q2. 취소 예외를 무시하면 안 되는 이유는 무엇인가요?**

<details>
<summary>답</summary>

상위 프레임워크가 연결 중단을 알 수 없고 정리 시점도 불명확해질 수 있습니다.

</details>

**Q3. 모든 조각을 list에 모아야 하나요?**

<details>
<summary>답</summary>

표시만 필요하면 즉시 흘려보내고, 저장·완료 검사에 필요한 경우에만 제한적으로 누적합니다.

</details>

## 한 줄 정리

> async generator는 비동기 대기와 조각 전송 사이를 연결하는 파이프라인의 기본 단위입니다.
