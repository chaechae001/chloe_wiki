# API 키와 서버 보안

> LLM API 키는 브라우저에 전달하지 않고 서버가 대신 외부 API를 호출하도록 설계합니다.

`API key` · `environment variable` · `secret` · `backend proxy` · `least privilege`

## 핵심요약

- API 키는 비밀번호처럼 취급하고 저장소·노트북·브라우저 코드에 넣지 않습니다.
- 클라이언트는 FastAPI를 호출하고, FastAPI만 LLM 제공자와 통신합니다.
- `.env`는 로컬 편의 수단이며 배포 환경에서는 비밀 관리 기능을 사용합니다.
- 로그와 오류 응답에서도 키, Authorization header, 원문 예외를 제거합니다.
- 노출이 의심되는 키는 숨기는 것만으로 끝내지 않고 폐기 후 재발급합니다.

## 1. 안전한 호출 경계

```text
Browser / App → POST /chat → FastAPI → LLM API
                              └─ API key 보관
```

브라우저 번들에 넣은 값은 사용자가 확인할 수 있습니다. 따라서 공개 클라이언트가 공급자 API를 직접 호출하도록 만들지 않고, 인증·사용량 제한·프롬프트 정책을 적용하는 서버를 사이에 둡니다.

<details>
<summary>답</summary>

백엔드 경계는 키를 숨기는 것뿐 아니라 사용자별 인증, 요청 한도, 감사 로그와 비용 통제를 한곳에 모읍니다.

</details>

## 2. 환경 변수로 설정 분리

```python
import os
from openai import OpenAI

api_key = os.environ["OPENAI_API_KEY"]
model = os.environ["OPENAI_MODEL"]
client = OpenAI(api_key=api_key)
```

`.env.example`에는 변수 이름만 기록하고 실제 값은 비워 둡니다. `.env`는 `.gitignore`에 추가해도 이미 커밋한 비밀을 되돌려 주지는 않으므로, 커밋 기록에 들어갔다면 키를 교체해야 합니다.

## 3. 호환 게이트웨이 설정

OpenAI 호환 서비스를 쓸 때만 문서에서 받은 `base_url`을 환경 변수로 분리합니다. 신뢰할 수 없는 URL로 키를 보내지 않도록 host allowlist를 둘 수 있습니다.

```python
base_url = os.getenv("LLM_BASE_URL")
client = OpenAI(api_key=api_key, base_url=base_url) if base_url else OpenAI(api_key=api_key)
```

## 코드로 보기 — 설정 검증

### 코드 목적

애플리케이션 시작 시 필수 설정 누락을 빠르게 발견합니다.

### 코드 흐름

1. 환경 변수를 읽습니다.
2. 빈 값이면 시작을 중단합니다.
3. 실제 비밀은 출력하지 않습니다.

### 실행 결과 해석

설정 누락이 요청 시점의 500 오류가 아니라 배포 시작 단계에서 드러납니다.

### 실무 연결

운영에서는 secret manager, 키 회전, 최소 권한과 접근 감사를 함께 적용합니다.

## 직접 해보기

1. `.env.example`에 필요한 변수 이름만 작성하세요.
2. 키를 출력하지 않는 설정 점검 함수를 만드세요.
3. 브라우저 직접 호출이 위험한 이유 두 가지를 적으세요.

<details>
<summary>정답 보기</summary>

1. `OPENAI_API_KEY=`와 `OPENAI_MODEL=`처럼 값 없이 구조만 공유합니다.
2. 존재 여부만 검사하고 오류에는 변수 이름만 포함합니다.
3. 키 탈취와 사용량·비용 통제 상실이 대표적입니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| `.env` vs secret manager | 로컬 파일 기반 설정과 배포 플랫폼의 암호화된 비밀 저장소입니다. |
| 숨김 vs 폐기 | 화면에서 가리는 것과 유출된 자격 증명을 무효화하는 것은 다릅니다. |
| API key vs 사용자 인증 | 공급자 호출 권한과 내 서비스 사용자 신원 확인은 별개입니다. |

## 연결되는 개념

- 다음: [Python SDK와 Responses API](02-python-sdk-and-responses-api.md)
- 함께 보면 좋은 키워드: `CORS`, `rate limiting`, `secret rotation`

## 셀프 체크

- [ ] 키를 클라이언트 코드에 두면 안 되는 이유를 설명한다.
- [ ] 실제 값 없는 환경 변수 예시를 만든다.
- [ ] `.gitignore`의 한계를 안다.
- [ ] 노출된 키의 교체 절차를 안다.
- [ ] 서비스 사용자 인증과 공급자 키를 구분한다.

### 복습 질문 및 답변

**Q1. `.env`를 ignore하면 키가 완전히 안전한가요?**

<details>
<summary>답</summary>

아닙니다. 화면 공유, 로그, 백업, 이전 커밋 등 다른 경로로 노출될 수 있습니다.

</details>

**Q2. 키 앞부분만 로그에 남겨도 되나요?**

<details>
<summary>답</summary>

가능하면 키 자체를 전혀 기록하지 않고 내부 식별자나 키 별칭을 사용합니다.

</details>

**Q3. 유출 의심 시 첫 조치는 무엇인가요?**

<details>
<summary>답</summary>

해당 키를 폐기·회전하고 사용 기록을 확인한 뒤 노출 경로를 제거합니다.

</details>

## 한 줄 정리

> LLM 키는 서버 경계 안에 두고, 노출 가능성이 생기면 즉시 교체합니다.
