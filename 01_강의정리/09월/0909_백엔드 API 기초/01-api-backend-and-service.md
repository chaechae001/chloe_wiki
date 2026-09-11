# API, Backend, Service의 역할

웹 화면의 버튼 하나가 실제 기능이 되려면 클라이언트의 요청을 서버가 해석하고, 내부 로직을 실행한 뒤, 다시 이해 가능한 응답을 돌려줘야 합니다. API는 이 왕복에 필요한 약속입니다.

**핵심 키워드:** API, client, server, backend, service

## API는 경계의 계약이다

API는 프로그램끼리 어떤 입력을 보내고 어떤 출력을 받을지 정한 인터페이스입니다. 클라이언트는 서버 내부 구현을 몰라도 메서드, 경로, 데이터 형식을 맞추면 기능을 사용할 수 있습니다.

```text
사용자 동작 → Client → HTTP Request → API Endpoint
→ 입력 검증 → Service → Response → Client 화면
```

클라이언트와 서버는 기기의 종류가 아니라 한 통신에서 맡는 역할입니다. 브라우저뿐 아니라 Python 스크립트도 요청을 보내면 클라이언트입니다.

## Backend와 Service

Backend는 요청이 들어오는 경계부터 데이터 검증, 인증, 서비스 호출, 응답 변환까지 서버 측 흐름을 포괄합니다. Service는 그 안에서 주문 계산이나 문서 요약처럼 실제 업무 규칙을 담당합니다.

```python
def summarize_service(text: str) -> str:
    return text[:40] + ("..." if len(text) > 40 else "")

def summarize_handler(payload: dict) -> tuple[dict, int]:
    text = payload.get("text")
    if not isinstance(text, str) or not text.strip():
        return {"error": {"code": "INVALID_TEXT"}}, 400

    result = summarize_service(text.strip())
    return {"data": {"summary": result}}, 200
```

handler가 HTTP 세부사항과 입력 검증을 담당하고 service가 업무 로직을 맡으면, service를 웹 서버 없이도 단위 테스트하기 쉽습니다.

## 계약을 안정적으로 유지하기

내부 알고리즘은 바뀔 수 있지만 요청·응답 계약이 유지되면 클라이언트 변경을 줄일 수 있습니다. 반대로 필드 이름이나 타입을 예고 없이 바꾸면 서버가 정상이어도 클라이언트가 깨집니다.

| 계층 | 주된 책임 | 변경 예 |
|---|---|---|
| Endpoint | 메서드·경로 연결 | `POST /summaries` |
| Validation | 입력 타입·규칙 확인 | 빈 문자열 거부 |
| Service | 실제 업무 처리 | 요약 알고리즘 변경 |
| Response mapping | 상태 코드·본문 구성 | 오류 코드 표준화 |

## 직접 해보기

1. Python 배치 프로그램이 외부 API를 호출할 때 어떤 역할인가요?
2. 입력 검증을 service 밖에 두면 얻는 장점을 설명하세요.
3. 챗봇 API의 endpoint와 service 책임을 나누어 보세요.

<details>
<summary>정답 보기</summary>

1. 요청을 보내므로 클라이언트 역할입니다.
2. HTTP 형식과 업무 로직을 분리해 service 재사용과 단위 테스트가 쉬워집니다.
3. endpoint는 인증·검증·응답 변환을, service는 모델 호출과 결과 후처리를 담당하도록 나눌 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| API vs 구현 | 외부에 공개한 계약 vs 계약을 수행하는 내부 코드 |
| Backend vs Service | 서버 측 전체 처리 영역 vs 핵심 업무 로직 |
| Client vs Server | 요청하는 역할 vs 요청을 받아 응답하는 역할 |

## 연결되는 개념

- 다음: [HTTP 요청과 응답 메시지](02-http-request-response.md)
- 함께 볼 키워드: `interface`, `endpoint`, `separation of concerns`

## 셀프 체크

- [ ] API를 프로그램 간 계약으로 설명한다.
- [ ] client와 server를 역할로 구분한다.
- [ ] backend와 service의 책임을 나눈다.
- [ ] handler에서 검증이 필요한 이유를 말한다.
- [ ] 계약 변경이 클라이언트에 미치는 영향을 이해한다.

### 복습 질문 및 답변

**Q1. 서버 내부 구현을 클라이언트가 알아야 하나요?**

<details>
<summary>답</summary>

아닙니다. 공개된 API 계약만 지키면 되고, 내부 구현은 계약을 깨지 않는 범위에서 바뀔 수 있습니다.

</details>

**Q2. 모든 로직을 endpoint 함수에 넣으면 왜 불리한가요?**

<details>
<summary>답</summary>

HTTP 처리와 업무 규칙이 결합되어 테스트, 재사용, 오류 원인 분리가 어려워집니다.

</details>

**Q3. API 문서는 왜 계약의 일부인가요?**

<details>
<summary>답</summary>

요청 필드, 타입, 응답, 오류 조건을 클라이언트가 예측할 수 있게 해 독립적인 개발을 가능하게 하기 때문입니다.

</details>

## 한 줄 정리

> API는 클라이언트와 서버의 약속이며, Backend는 경계를 관리하고 Service는 실제 업무를 수행합니다.
