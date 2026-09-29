# 실무형 레이어드 API 구조

> Router, Service, Client를 분리하면 HTTP 규칙, 비즈니스 로직, 외부 LLM 호출의 변경 이유가 섞이지 않습니다.

`Router` · `Service` · `Client` · `Schema` · `dependency direction`

## 핵심요약

- 호출 흐름은 `Router → Service → Client` 한 방향으로 유지합니다.
- Router는 HTTP 입력·출력, Service는 업무 규칙, Client는 외부 API를 담당합니다.
- Schema는 여러 레이어가 공유하는 입력·출력 계약입니다.
- 설정과 로깅은 `core`, 프롬프트는 별도 콘텐츠 영역에 둡니다.
- 파일 수보다 변경 이유와 의존성 방향이 분리되었는지가 중요합니다.

## 1. 한 파일이 커질 때 생기는 문제

라우팅, 검증, 프롬프트 조립, SDK 호출, 캐싱, 오류 처리까지 한 함수에 넣으면 어느 수정이 다른 기능을 깨뜨렸는지 찾기 어렵습니다. 같은 호출 코드가 여러 endpoint에 복제되고 테스트도 실제 네트워크에 의존하게 됩니다.

<details>
<summary>답</summary>

코드가 길어서가 아니라 서로 다른 변경 이유가 한곳에 섞여 결합도가 높아지는 것이 핵심 문제입니다.

</details>

## 2. 레이어의 책임

| 레이어 | 책임 | 알지 않아도 되는 것 |
|---|---|---|
| Router | HTTP method, path, request·response | SDK 내부 형식 |
| Service | 프롬프트 조립, 정책, 캐시, 후처리 | HTTP framework 세부사항 |
| Client | SDK 호출, timeout, provider 응답 변환 | URL과 status code |
| Schema | 입력·출력 타입과 validation | 실제 처리 방식 |
| Core | 환경 변수, 공통 logging 설정 | 기능별 정책 |

```text
HTTP request → Router → Service → Client → External LLM
```

아래 레이어가 위 레이어를 import하지 않도록 하면 외부 provider 교체와 단위 테스트가 쉬워집니다.

## 3. 기본 디렉터리

```text
app/
├── main.py
├── core/
├── schemas/
├── routers/
├── services/
├── clients/
└── prompts/
```

`prompts/`는 호출 순서의 레이어라기보다 버전 관리되는 콘텐츠 영역입니다. 작은 프로젝트라면 합칠 수 있지만 책임 경계는 유지합니다.

## 직접 해보기

1. LLM SDK 호출을 어느 레이어에 둘지 설명하세요.
2. 요청 body 검증 모델의 위치를 정하세요.
3. Service가 FastAPI의 `HTTPException`을 직접 쓰지 않게 바꿔 보세요.

<details>
<summary>정답 보기</summary>

SDK 호출은 Client, Pydantic 계약은 Schema에 둡니다. Service는 도메인 예외를 발생시키고 Router가 이를 HTTP 응답으로 변환하면 framework 의존성이 줄어듭니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| 레이어 vs 폴더 | 책임·의존성 규칙과 파일 배치 방식입니다. |
| Service vs Client | 업무 정책과 외부 시스템 통신입니다. |
| 단순함 vs 한 파일 | 구조가 단순한 것과 책임이 뒤섞인 것은 다릅니다. |

## 연결되는 개념

- 다음: [설정과 Schema 설계](02-configuration-and-schemas.md)
- 함께 보기: [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] Router·Service·Client 책임을 구분한다.
- [ ] 의존성 방향을 설명한다.
- [ ] HTTP 세부사항을 Service에서 분리한다.
- [ ] Provider SDK를 Client에 한정한다.
- [ ] 프로젝트 규모에 맞춰 구조를 조절한다.

### 복습 질문 및 답변

**Q1. Router가 프롬프트를 직접 조립해도 되나요?**

<details>
<summary>답</summary>

동작은 하지만 비즈니스 정책이 HTTP 계층에 섞입니다. 프롬프트 조립은 Service에 두는 편이 변경과 테스트에 유리합니다.

</details>

**Q2. 레이어를 나누면 파일만 많아지는 것 아닌가요?**

<details>
<summary>답</summary>

작은 기능에는 과할 수 있습니다. 변경 이유와 테스트 경계가 실제로 다를 때 분리하는 것이 목적입니다.

</details>

**Q3. Client가 Pydantic request 전체를 받아도 되나요?**

<details>
<summary>답</summary>

Provider 호출에 필요한 값만 넘기면 Client가 웹 요청 계약에 덜 결합됩니다.

</details>

## 한 줄 정리

> 레이어드 구조는 코드를 나누는 기법이 아니라 변경 이유와 의존성 방향을 통제하는 규칙입니다.
