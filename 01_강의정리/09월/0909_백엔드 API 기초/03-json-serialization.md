# JSON과 데이터 직렬화

JSON은 HTTP API에서 데이터를 주고받을 때 널리 쓰이는 텍스트 형식입니다. 단순한 문자열처럼 보여도 타입과 구조를 지켜야 서버와 클라이언트가 같은 의미로 해석할 수 있습니다.

**핵심 키워드:** JSON, serialization, deserialization, schema, validation

## JSON의 값과 구조

JSON 값은 object, array, string, number, boolean, `null`로 구성됩니다. object의 key는 큰따옴표로 감싼 문자열이어야 합니다.

```json
{
  "message": "작업을 접수했습니다.",
  "priority": 2,
  "active": true,
  "tags": ["api", "backend"],
  "metadata": null
}
```

Python의 `None`, `True`, `False`는 JSON에서 각각 `null`, `true`, `false`가 됩니다. 작은 표기 차이가 파싱 실패를 만들 수 있습니다.

<details>
<summary>답</summary>

JSON은 JavaScript object literal과 비슷하지만 동일하지 않습니다. JSON에서는 key와 문자열에 큰따옴표를 사용하며, 주석이나 함수 같은 값은 허용하지 않습니다.

</details>

## 직렬화와 역직렬화

직렬화는 메모리의 객체를 전송·저장 가능한 형식으로 바꾸는 과정이고, 역직렬화는 받은 데이터를 다시 프로그램의 값으로 복원하는 과정입니다.

```python
import json

payload = {"topic": "HTTP", "completed": False}
text = json.dumps(payload, ensure_ascii=False)
restored = json.loads(text)
```

HTTP client 라이브러리의 `json=` 옵션은 보통 직렬화와 `Content-Type: application/json` 설정을 함께 처리합니다. 반면 문자열을 직접 `data=`로 보내면 header와 인코딩을 개발자가 확인해야 합니다.

<details>
<summary>답</summary>

직렬화가 성공했다고 데이터가 유효하다는 뜻은 아닙니다. JSON 문법은 맞더라도 필수 field가 없거나 허용 범위를 벗어나면 schema 또는 업무 규칙 검증에서 실패할 수 있습니다.

</details>

## Schema로 계약 명시하기

Schema는 필요한 field, 타입, 기본값, 제약 조건을 명시합니다. 서버는 handler에 들어오기 전에 잘못된 입력을 걸러낼 수 있고, 클라이언트는 요청 형식을 예측할 수 있습니다.

```python
from pydantic import BaseModel, Field

class MessageCreate(BaseModel):
    text: str = Field(min_length=1, max_length=500)
    priority: int = Field(default=1, ge=1, le=5)
```

`text`가 비어 있거나 `priority`가 범위를 벗어나면 검증 오류로 처리할 수 있습니다. Schema는 API 문서와 테스트의 기준으로도 활용됩니다.

<details>
<summary>답</summary>

Optional field는 생략 가능한 field이고, nullable field는 명시적으로 `null`을 받을 수 있는 field입니다. 두 개념은 같지 않으므로 API 계약에서 구분해야 합니다.

</details>

## 오류를 계층별로 구분하기

| 계층 | 예시 | 대응 |
|---|---|---|
| JSON 문법 | 닫는 괄호 누락 | 파싱 오류 반환 |
| Schema 타입 | 숫자 자리에 object | field별 검증 오류 반환 |
| 업무 규칙 | 잔여 수량보다 큰 주문 | 충돌 또는 도메인 오류 반환 |
| 참조 관계 | 존재하지 않는 사용자 ID | not found 오류 반환 |

모든 오류를 500으로 처리하면 클라이언트가 수정 가능한 요청인지 서버 장애인지 알 수 없습니다. 오류가 발생한 계층에 맞는 상태 코드와 메시지를 사용합니다.

<details>
<summary>답</summary>

오류 응답에는 사람이 읽을 message와 프로그램이 분기할 안정적인 code를 함께 두면 좋습니다. 단, stack trace나 내부 경로 같은 구현 정보는 외부 응답에 노출하지 않습니다.

</details>

## 비교: 느슨한 object와 명시적 schema

| 구분 | 자유로운 object | Schema 기반 모델 |
|---|---|---|
| 시작 속도 | 빠름 | 정의 작업 필요 |
| 오류 발견 | 실행 중 늦게 발견 | 경계에서 조기 발견 |
| 문서화 | 별도 관리 필요 | 자동화하기 쉬움 |
| 변경 영향 | 파악하기 어려움 | 계약 변화 추적 용이 |

## 실습

1. 제목과 공개 여부, 태그 목록을 담은 유효한 JSON을 작성하세요.
2. `{"count": "many"}`가 문법과 schema 관점에서 각각 유효할 수 있는지 설명하세요.
3. 필수 field 누락 오류의 응답 구조를 설계하세요.

<details>
<summary>정답 보기</summary>

예: `{"title":"API 노트","public":true,"tags":["http"]}`. `count` 예시는 JSON 문법상 유효하지만 schema가 숫자를 요구하면 검증에 실패합니다. 오류 응답은 `{"error":{"code":"VALIDATION_ERROR","message":"입력값을 확인해 주세요.","fields":{"count":"필수 값입니다."}}}`처럼 구성할 수 있습니다.

</details>

## 셀프 체크

- [ ] JSON에서 사용할 수 있는 값의 종류를 말할 수 있다.
- [ ] 직렬화와 역직렬화를 구분할 수 있다.
- [ ] 문법 오류와 schema 오류를 구별할 수 있다.
- [ ] optional과 nullable의 차이를 설명할 수 있다.
- [ ] 내부 정보를 노출하지 않는 오류 구조를 설계할 수 있다.

이전 글: [HTTP 요청과 응답 메시지](02-http-request-response.md) · 다음 글: [REST 리소스와 Endpoint 설계](04-rest-resource-design.md)
