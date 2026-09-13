# TestClient로 API 검증하기

> 눈으로 한 번 눌러본 API보다 같은 조건을 언제든 재현하는 테스트가 변경에 강합니다.

`TestClient` · `pytest` · `assert` · `validation error` · `notebook state`

## 핵심요약

- TestClient는 실제 socket 연결 없이 FastAPI 앱을 호출합니다.
- 상태 코드와 JSON body를 assertion으로 함께 검증합니다.
- 정상 경로와 검증 실패 경로를 모두 테스트합니다.
- Notebook에서는 이전 변수와 출력이 남아 결과를 오해할 수 있습니다.
- 독립적인 테스트 함수는 숨은 실행 순서 의존성을 줄입니다.

## 1. 첫 API 테스트

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_status() -> None:
    response = client.get("/status")
    assert response.status_code == 200
    assert response.json() == {"state": "ready"}
```

TestClient는 HTTPX 기반 인터페이스를 사용합니다. Query parameter는 `params=`, JSON body는 `json=`, header는 `headers=`에 dictionary로 전달합니다.

<details>
<summary>답</summary>

응답을 출력만 하면 사람이 다시 확인해야 하지만 assertion은 기대와 실제가 다를 때 자동으로 실패합니다. 회귀 테스트에는 assertion이 핵심입니다.

</details>

## 2. 정상과 실패를 쌍으로 검사하기

```python
def test_book_id_validation() -> None:
    ok = client.get("/books/7")
    invalid = client.get("/books/not-a-number")

    assert ok.status_code == 200
    assert ok.json()["book_id"] == 7
    assert invalid.status_code == 422
    assert invalid.json()["detail"][0]["loc"] == ["path", "book_id"]
```

상태 코드만 검사하면 잘못된 body를 놓칠 수 있고, body만 검사하면 잘못된 상태 코드를 놓칠 수 있습니다. API 계약은 둘을 함께 포함합니다.

<details>
<summary>답</summary>

입력 검증 실패는 route 함수가 실행되기 전에 발생할 수 있습니다. 따라서 service mock의 호출 여부까지 확인하면 잘못된 입력이 업무 로직에 전달되지 않았는지도 검증할 수 있습니다.

</details>

## 3. Notebook의 숨은 상태 피하기

Notebook에서는 변수 이름을 잘못 쓰거나 셀을 순서와 다르게 실행해도 과거 값이 남아 있을 수 있습니다. 호출 결과를 새 변수에 저장하고 다른 변수의 응답을 출력하면 화면은 그럴듯하지만 실제 요청과 무관한 결과가 보입니다.

```python
def test_search_parameters() -> None:
    response = client.get(
        "/search",
        params={"keyword": "python", "limit": 3},
    )
    assert response.status_code == 200
    assert response.json() == {"keyword": "python", "limit": 3}
```

Kernel restart 후 위에서 아래로 실행하고, 최종적으로 pytest 함수로 옮기면 재현성을 높일 수 있습니다. 함수 이름만 적는 것과 `function(argument)`로 실제 호출하는 것도 구분해야 합니다.

## 코드로 보기 — body 검증 테스트

```python
def test_summary_body_required() -> None:
    success = client.post("/summaries", json={"text": "short note"})
    failure = client.post("/summaries", json={})

    assert success.status_code == 200
    assert "summary" in success.json()
    assert failure.status_code == 422
```

### 코드 목적

정상 body와 필수 field 누락을 하나의 계약 테스트 흐름으로 비교합니다.

### 코드 흐름

1. 정상 JSON을 전송합니다.
2. 성공 상태와 핵심 field를 확인합니다.
3. 빈 JSON을 전송합니다.
4. 자동 검증 실패 상태를 확인합니다.

### 실행 결과 해석

두 assertion 묶음이 모두 통과해야 성공 처리와 입력 차단이 함께 보장됩니다.

### 실무 연결

CI에서 테스트를 실행하면 endpoint 수정이 client 계약을 깨는 시점을 배포 전에 발견할 수 있습니다.

## 직접 해보기

1. Query 기본값이 적용되는 테스트를 작성하세요.
2. JSON body의 필수 field 누락이 422인지 확인하세요.
3. 과거 Notebook 변수 때문에 잘못된 결과가 보이는 상황의 방지책을 설명하세요.

<details>
<summary>정답 보기</summary>

1. Query를 생략해 호출한 뒤 기본값이 담긴 JSON을 assertion합니다.
2. `client.post(path, json={})`의 상태 코드를 확인합니다.
3. 변수명을 일관되게 쓰고, kernel restart 후 전체 실행하며, 독립적인 pytest 함수와 assertion으로 옮깁니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Print vs assert | 사람이 읽는 출력 vs 기대 조건을 자동 판정하는 검사 |
| Function reference vs call | 함수 객체를 가리킴 vs 괄호와 인자로 실제 실행함 |
| Notebook cell test vs pytest | 탐색에 편리하지만 상태가 남음 vs 독립성과 반복성을 강조함 |

## 연결되는 개념

- 이전: [OpenAPI와 자동 API 문서](03-openapi-and-automatic-docs.md)
- 다음: [텍스트 처리 Service와 Endpoint](05-text-service-and-endpoint.md)
- 함께 보면 좋은 키워드: `fixture`, `mock`, `CI`

## 셀프 체크

- [ ] TestClient의 목적을 설명할 수 있다.
- [ ] Query와 JSON body를 테스트 요청에 전달할 수 있다.
- [ ] 상태 코드와 body를 함께 assertion할 수 있다.
- [ ] 정상·실패 사례를 쌍으로 설계할 수 있다.
- [ ] Notebook의 숨은 상태 문제를 예방할 수 있다.

### 복습 질문 및 답변

**Q1. TestClient는 별도 서버 process가 필요한가요?**

<details>
<summary>답</summary>

일반적인 TestClient 테스트는 별도 socket server를 띄우지 않고 애플리케이션 코드와 직접 통신합니다.

</details>

**Q2. 테스트 함수가 `async def`여야 하나요?**

<details>
<summary>답</summary>

동기 TestClient를 사용하는 일반 pytest 테스트는 보통 `def`로 작성합니다. 테스트 자체에서 비동기 함수를 직접 await해야 한다면 별도 async test 구성이 필요합니다.

</details>

**Q3. 200만 검사하면 어떤 회귀를 놓칠 수 있나요?**

<details>
<summary>답</summary>

Field 이름이나 값이 잘못되거나 민감 field가 추가되는 등 response body 계약의 변화를 놓칠 수 있습니다.

</details>

## 한 줄 정리

> TestClient와 assertion은 성공·실패 응답을 반복 가능한 계약 검사로 바꾸며 Notebook의 실행 순서 착시도 줄입니다.
