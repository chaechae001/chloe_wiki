# FastAPI 앱 구조와 서버 실행

> Python 함수에 URL 규칙을 연결하면 작은 함수가 웹에서 호출할 수 있는 API endpoint가 됩니다.

`FastAPI` · `ASGI` · `Uvicorn` · `path operation` · `reload`

## 핵심요약

- `FastAPI()` 인스턴스가 API 애플리케이션의 중심입니다.
- 데코레이터는 HTTP method와 path를 함수에 연결합니다.
- 함수가 반환한 Python 값은 JSON 응답으로 직렬화됩니다.
- Uvicorn은 ASGI 애플리케이션을 실제 요청과 연결하는 서버입니다.
- 개발용 reload와 운영 배포 설정은 목적이 다릅니다.

## 1. 앱과 path operation

FastAPI에서 endpoint는 데코레이터와 함수의 조합입니다. 아래 코드는 기존 예제를 복제하지 않고 구조 확인용으로 새로 작성한 최소 예제입니다.

```python
from fastapi import FastAPI

app = FastAPI(title="Learning API", version="1.0.0")

@app.get("/status")
def read_status() -> dict[str, str]:
    return {"state": "ready"}
```

`@app.get("/status")`는 GET 요청과 `/status`를 바로 아래 함수에 연결합니다. 반환된 dictionary는 JSON body로 변환되고, 별도 상태 코드를 지정하지 않으면 성공 응답은 보통 200입니다.

<details>
<summary>답</summary>

데코레이터의 path와 method가 요청을 받을 위치를 정하고, 함수는 그 요청을 처리할 로직을 정의합니다. 둘을 합쳐 path operation이라고 부릅니다.

</details>

## 2. Uvicorn으로 실행하기

```bash
uvicorn main:app --reload
```

`main`은 모듈 이름, `app`은 그 모듈 안의 FastAPI 인스턴스 이름입니다. `--reload`는 파일 변경을 감지해 개발 서버를 다시 시작하므로 학습과 로컬 개발에 편리하지만 운영 환경의 기본 설정으로 사용하지 않습니다.

```text
HTTP client → Uvicorn → FastAPI app → path operation → JSON response
```

<details>
<summary>답</summary>

명령의 `main:app`과 실제 파일·변수 이름이 다르면 Uvicorn이 애플리케이션을 import하지 못합니다. 실행 위치와 Python module path도 함께 확인해야 합니다.

</details>

## 3. 동기 함수와 비동기 함수

FastAPI는 `def`와 `async def` path operation을 모두 지원합니다. 비동기 라이브러리를 `await`해야 한다면 `async def`가 자연스럽고, blocking 작업은 실행 방식을 별도로 고려해야 합니다.

```python
@app.get("/welcome/{name}")
async def welcome(name: str) -> dict[str, str]:
    return {"message": f"Welcome, {name}"}
```

`async def`라는 표기만으로 CPU 작업이 빨라지는 것은 아닙니다. 핵심은 대기 시간이 있는 I/O를 막지 않는 방식으로 연결하는 것입니다.

## 코드로 보기 — 서버 시작점 읽기

### 코드 목적

앱 메타데이터와 상태 확인 endpoint를 구성합니다.

### 코드 흐름

1. FastAPI를 import합니다.
2. 앱 인스턴스를 만듭니다.
3. method와 path를 함수에 등록합니다.
4. Uvicorn이 앱을 import해 요청을 전달합니다.

### 실행 결과 해석

`GET /status`에서 200과 `{"state":"ready"}`가 오면 routing과 기본 직렬화가 동작한 것입니다.

### 실무 연결

상태 확인 endpoint는 배포 환경의 health check와 장애 진단의 출발점이 됩니다.

## 직접 해보기

1. `POST /reports`를 처리할 빈 endpoint를 선언해 보세요.
2. `server.py`의 `api` 인스턴스를 reload 모드로 실행하는 명령을 작성하세요.
3. 운영 환경에서 `--reload` 사용을 피해야 하는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. `@app.post("/reports")`와 처리 함수를 조합할 수 있습니다.
2. `uvicorn server:api --reload`입니다.
3. 파일 감시와 재시작은 개발 편의 기능이라 추가 자원을 사용하며, 안정적인 운영 process 관리 방식과 목적이 다릅니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| FastAPI vs Uvicorn | FastAPI는 애플리케이션 framework이고 Uvicorn은 ASGI server입니다. |
| `def` vs `async def` | 동기 함수와 coroutine 함수이며 사용하는 I/O 방식에 맞춰 선택합니다. |
| reload vs restart policy | reload는 코드 감지용 개발 기능, restart policy는 장애 복구용 운영 정책입니다. |

## 연결되는 개념

- 다음: [요청 파라미터와 Pydantic 모델](02-request-parameters-and-pydantic.md)
- 함께 보면 좋은 키워드: `routing`, `serialization`, `event loop`

## 셀프 체크

- [ ] FastAPI 앱 인스턴스의 역할을 설명할 수 있다.
- [ ] 데코레이터와 함수가 endpoint를 만드는 과정을 이해한다.
- [ ] `module:object` 실행 표기를 해석할 수 있다.
- [ ] FastAPI와 Uvicorn을 구분할 수 있다.
- [ ] 개발용 reload의 용도를 설명할 수 있다.

### 복습 질문 및 답변

**Q1. Python dictionary가 HTTP 응답으로 어떻게 전달되나요?**

<details>
<summary>답</summary>

FastAPI가 반환값을 JSON과 호환되는 데이터로 직렬화하고, HTTP response body와 적절한 content type을 구성합니다.

</details>

**Q2. 같은 path에 GET과 POST를 각각 등록할 수 있나요?**

<details>
<summary>답</summary>

가능합니다. Endpoint는 path뿐 아니라 HTTP method까지 포함해 구분됩니다.

</details>

**Q3. `async def` 안에서 오래 걸리는 동기 I/O를 직접 실행하면 무엇이 문제인가요?**

<details>
<summary>답</summary>

Event loop를 막아 다른 요청의 처리가 지연될 수 있습니다. 비동기 client나 별도 thread·worker 같은 적절한 실행 방식을 선택해야 합니다.

</details>

## 한 줄 정리

> FastAPI는 요청 규칙과 Python 함수를 연결하고, Uvicorn은 그 애플리케이션을 네트워크 요청에 노출합니다.
