# 스트리밍 오류·취소·결과 점검

> 스트림은 status 200만으로 성공을 판단할 수 없으므로, 완료 신호·본문·로그·취소 정리를 함께 검증합니다.

`timeout` · `CancelledError` · `finally` · `completion` · `contract test`

## 핵심요약

- stream 시작 전 오류와 시작 후 오류를 구분합니다.
- timeout은 서버와 클라이언트 양쪽에 명시합니다.
- 클라이언트 중단 시 `CancelledError`는 정리 후 다시 raise합니다.
- `finally`는 정상 완료·오류·취소 모두에서 정리를 수행합니다.
- 테스트는 status뿐 아니라 data sequence와 종료 상태를 확인합니다.

## 1. 오류 분류

| 시점 | 예시 | 대응 |
|---|---|---|
| 시작 전 | 잘못된 입력·인증 구성 | 명확한 HTTP 오류 |
| 전송 중 | upstream timeout·연결 단절 | error event 또는 종료 protocol |
| 완료 후 | 빈 결과·불완전 결과 | completion marker와 결과 검사 |
| 클라이언트 중단 | 탭 이동·읽기 중단 | cancel 전파와 resource cleanup |

<details>
<summary>답</summary>

stream이 이미 시작되면 응답 status와 header가 클라이언트에 전송됐을 수 있으므로 본문 protocol과 서버 로그가 중요해집니다.

</details>

## 2. 취소 안전 패턴

```python
import asyncio

async def generate():
    try:
        async for piece in upstream():
            yield piece
    except asyncio.CancelledError:
        # 비밀을 제외한 취소 metric·정리
        raise
    except TimeoutError:
        yield "event: error\ndata: timeout\n\n"
    finally:
        await close_upstream_if_needed()
```

취소를 정상 완료처럼 숨기지 말고, connection·file·task 같은 리소스를 finally에서 정리합니다.

## 3. 결과 점검

성공 검증에는 status, 순서대로 받은 text/event, 완료 marker, 첫 조각 도착 시간, 서버 request ID를 함께 봅니다. 실제 모델 문장을 고정 비교하기보다 fake upstream으로 오류·완료 계약을 테스트합니다.

## 직접 해보기

1. 서버와 클라이언트 timeout을 각각 설정하세요.
2. fake upstream timeout의 error event를 검증하세요.
3. 3개 조각 후 읽기를 중단해 finally 실행을 확인하세요.

<details>
<summary>정답 보기</summary>

timeout 값은 서비스 요구에 맞춰 상한을 정하고, fake generator가 예외를 낼 때 error event와 cleanup metric을 assertion합니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| timeout vs cancellation | 시간 제한 초과와 소비자의 의도적 연결 종료입니다. |
| status 200 vs 완료 | HTTP 시작 성공과 모델 생성의 정상 완료입니다. |
| finally vs except | 모든 종료 경로 정리와 특정 오류 처리입니다. |

## 연결되는 개념

- 이전: [chat stream endpoint 설계](05-chat-stream-endpoint.md)
- 함께 보기: [전체 개요](OVERVIEW.md)

## 셀프 체크

- [ ] 시작 전·후 오류를 구분한다.
- [ ] 양쪽 timeout을 설정한다.
- [ ] CancelledError를 re-raise한다.
- [ ] finally에서 resource를 정리한다.
- [ ] 종료 marker와 stream sequence를 테스트한다.

### 복습 질문 및 답변

**Q1. 200이면 모델 생성도 성공한 것인가요?**

<details>
<summary>답</summary>

아닙니다. body가 시작된 뒤 upstream 오류나 빈 결과가 생길 수 있습니다.

</details>

**Q2. client timeout만 있어도 충분한가요?**

<details>
<summary>답</summary>

서버 작업과 자원 점유를 제한하려면 upstream 호출에도 timeout과 취소 처리가 필요합니다.

</details>

**Q3. 왜 fake upstream test가 유용한가요?**

<details>
<summary>답</summary>

네트워크·비용·모델 변동 없이 오류와 완료 protocol을 반복해서 검증할 수 있습니다.

</details>

## 한 줄 정리

> 스트리밍 성공은 HTTP status가 아니라 조각, 종료 신호, 취소 정리까지 확인해야 합니다.
