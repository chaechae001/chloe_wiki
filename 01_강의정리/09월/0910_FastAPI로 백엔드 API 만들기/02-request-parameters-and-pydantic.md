# 요청 파라미터와 Pydantic 모델

> FastAPI는 함수 signature를 읽어 값의 출처와 검증 규칙을 결정합니다.

`path parameter` · `query parameter` · `request body` · `Pydantic` · `422`

## 핵심요약

- Path parameter는 특정 리소스를 식별합니다.
- Query parameter는 필터·검색·페이지 조건에 적합합니다.
- Pydantic 모델은 JSON request body의 구조와 타입을 정의합니다.
- 기본값이 있으면 선택 입력, 없으면 필수 입력이 됩니다.
- 검증 실패는 handler 실행 전에 구조화된 오류로 반환됩니다.

## 1. 값의 출처를 구분하는 규칙

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/books/{book_id}")
def read_book(
    book_id: int,
    keyword: str | None = None,
    limit: int = Query(default=10, ge=1, le=50),
) -> dict:
    return {"book_id": book_id, "keyword": keyword, "limit": limit}
```

Path에 이름이 포함된 `book_id`는 path parameter입니다. `str`, `int` 같은 단순 타입의 나머지 인자는 query parameter로 해석됩니다. 타입 힌트와 `Query` 제약은 변환, 검증, 자동 문서에 함께 쓰입니다.

<details>
<summary>답</summary>

`/books/abc`는 `book_id: int`로 변환할 수 없어 handler가 실행되기 전에 검증 실패 응답이 발생합니다. 단순한 타입 힌트가 런타임 요청 계약의 일부가 되는 사례입니다.

</details>

## 2. Request body 모델

```python
from pydantic import BaseModel, Field

class SummaryRequest(BaseModel):
    text: str = Field(min_length=1, max_length=1000)
    style: str = "brief"

@app.post("/summaries")
def create_summary(payload: SummaryRequest) -> dict[str, str]:
    return {"summary": payload.text[:40], "style": payload.style}
```

Pydantic 모델 타입의 인자는 request body로 해석됩니다. `text`는 필수이고 `style`은 기본값이 있어 생략할 수 있습니다. 현재 Pydantic에서는 모델을 dictionary로 바꿀 때 `model_dump()`를 사용할 수 있습니다.

<details>
<summary>답</summary>

`str | None`은 `None`을 허용한다는 타입 의미이고, 입력을 생략할 수 있는지는 기본값의 유무로 결정됩니다. Nullable과 optional을 같은 개념으로 보지 않아야 합니다.

</details>

## 3. 422 응답 읽기

검증 오류의 `loc`는 문제가 생긴 위치를 알려줍니다. 예를 들어 `['path', 'book_id']`는 path 값, `['body', 'text']`는 JSON body의 field 문제입니다. `msg`와 `type`은 원인을 자동 테스트에서 확인할 때 유용합니다.

| 위치 | 대표 원인 | 해결 방향 |
|---|---|---|
| path | 정수 변환 실패 | URL 식별자 확인 |
| query | 범위 초과 | 필터 값 제약 확인 |
| body | 필수 field 누락 | JSON key와 schema 확인 |

## 코드로 보기 — 세 입력을 한 endpoint에서 받기

```python
class ReviewCreate(BaseModel):
    comment: str = Field(min_length=1)

@app.post("/books/{book_id}/reviews")
def create_review(book_id: int, payload: ReviewCreate, notify: bool = False) -> dict:
    return {"book_id": book_id, "comment": payload.comment, "notify": notify}
```

### 코드 목적

식별자, JSON body, 선택 옵션을 하나의 요청에서 구분합니다.

### 코드 흐름

1. Router가 path에서 `book_id`를 읽습니다.
2. JSON body를 `ReviewCreate`로 검증합니다.
3. Query string에서 `notify`를 변환합니다.
4. 모든 검증이 성공한 뒤 함수가 실행됩니다.

### 실행 결과 해석

입력 하나라도 계약을 만족하지 않으면 업무 로직에 도달하지 않고 오류 위치가 포함된 422 응답을 받습니다.

### 실무 연결

입력 경계를 schema로 고정하면 endpoint마다 반복되는 수동 타입 검사를 줄이고 API 문서를 최신 코드와 맞출 수 있습니다.

## 직접 해보기

1. `/users/{user_id}`의 `user_id`를 정수로 받는 endpoint를 만드세요.
2. `page`가 1 이상인 query parameter를 선언하세요.
3. 제목은 필수, 공개 여부는 기본값이 `False`인 body 모델을 설계하세요.

<details>
<summary>정답 보기</summary>

1. Path와 함수 인자에 같은 이름을 쓰고 `user_id: int`로 선언합니다.
2. `page: int = Query(default=1, ge=1)`처럼 작성합니다.
3. `title: str`과 `published: bool = False`를 가진 `BaseModel`을 만들 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| Path vs query | 대상 식별 vs 조회 조건 조절 |
| Query vs body | URL의 간단한 조건 vs 구조화된 입력 데이터 |
| 타입 오류 vs 업무 오류 | 입력 변환·schema 실패 vs 유효한 입력에 대한 업무 규칙 실패 |

## 연결되는 개념

- 이전: [FastAPI 앱 구조와 서버 실행](01-fastapi-app-and-server.md)
- 다음: [OpenAPI와 자동 API 문서](03-openapi-and-automatic-docs.md)
- 함께 보면 좋은 키워드: `JSON Schema`, `validation`, `dependency injection`

## 셀프 체크

- [ ] Path, query, body 입력을 구분할 수 있다.
- [ ] 기본값과 필수 여부의 관계를 설명할 수 있다.
- [ ] Pydantic field 제약을 작성할 수 있다.
- [ ] 422 응답의 `loc`를 해석할 수 있다.
- [ ] 검증과 업무 로직의 경계를 설명할 수 있다.

### 복습 질문 및 답변

**Q1. Path의 이름과 함수 인자 이름이 달라도 되나요?**

<details>
<summary>답</summary>

기본 선언에서는 서로 일치해야 FastAPI가 해당 값을 함수 인자에 연결할 수 있습니다.

</details>

**Q2. Pydantic 모델을 쓰면 자동 문서에는 무엇이 추가되나요?**

<details>
<summary>답</summary>

Field 이름, 타입, 필수 여부, 기본값과 제약이 JSON Schema로 표현되어 요청 body 문서와 입력 UI에 반영됩니다.

</details>

**Q3. 422를 무조건 서버 오류로 보아도 되나요?**

<details>
<summary>답</summary>

아닙니다. FastAPI에서는 보통 요청 데이터가 선언된 schema를 만족하지 못했다는 뜻이므로 client 입력과 오류 위치를 먼저 확인합니다.

</details>

## 한 줄 정리

> 함수 signature와 Pydantic 모델은 FastAPI 요청의 위치, 타입, 필수 여부를 표현하는 실행 가능한 계약입니다.
