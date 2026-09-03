# MVC 기반 CRUD와 API 테스트

API가 커질수록 HTTP 처리, 비즈니스 규칙과 데이터 접근을 분리해야 변경과 테스트의 범위를 통제할 수 있습니다.

**핵심 키워드:** MVC, controller, model, CRUD, API test

## 핵심 내용

- Router는 메서드와 경로를 컨트롤러에 연결합니다.
- Controller는 HTTP 입력을 해석하고 서비스 결과를 응답으로 변환합니다.
- Model 또는 데이터 계층은 데이터 구조와 저장소 접근을 담당합니다.
- View가 없는 JSON API에서는 JSON 표현이 응답 View 역할을 할 수 있습니다.
- API 테스트는 성공뿐 아니라 검증 실패, 자원 없음과 시스템 오류를 함께 확인합니다.

## 계층 구조

```text
Request
  → Middleware
  → Router
  → Controller
  → Service
  → Repository / Model
  → Response
```

단순한 예제에서는 모든 계층이 필요하지 않지만, 역할은 구분해야 합니다. 컨트롤러가 데이터 저장 형식과 모든 규칙을 직접 알기 시작하면 테스트와 교체가 어려워집니다.

## CRUD 라우트 구성

```javascript
const router = require("express").Router();
const controller = require("./note-controller");

router.get("/", controller.list);
router.get("/:noteId", controller.getOne);
router.post("/", controller.create);
router.patch("/:noteId", controller.update);
router.delete("/:noteId", controller.remove);
```

- **목적:** 노트 자원의 CRUD 계약을 한 Router에서 확인합니다.
- **흐름:** Router 매칭 → 컨트롤러 입력 처리 → 서비스·모델 호출 → HTTP 응답입니다.
- **결과:** 경로 구조와 구현 책임이 분리됩니다.
- **실무 포인트:** 실제 저장소를 메모리 배열로 시작해도 동시성, ID, 데이터 지속성과 테스트 초기화 규칙을 명시합니다.

## Controller 예시

```javascript
async function create(req, res, next) {
  try {
    const note = await noteService.create({ title: req.body.title });
    return res.status(201).json(note);
  } catch (error) {
    return next(error);
  }
}
```

Controller는 Express 객체를 알고, 서비스는 가능한 한 HTTP 세부사항에서 독립적으로 유지합니다. 이렇게 하면 비즈니스 규칙을 빠른 단위 테스트로 검증할 수 있습니다.

## API 테스트 계획

| 범주 | 확인 예 |
|---|---|
| 성공 | 생성 후 201과 응답 스키마 |
| 입력 오류 | 필수 필드 누락 시 400 |
| 자원 없음 | 없는 ID 조회 시 404 |
| 권한 | 허용되지 않은 작업 시 401 또는 403 |
| 시스템 오류 | 안전한 500 응답과 서버 로그 |

Postman 같은 API 클라이언트로 요청을 탐색하고 예시를 문서화할 수 있습니다. 회귀 방지를 위해 핵심 계약은 자동 테스트에도 포함합니다. 환경 변수에는 base URL 같은 비밀이 아닌 설정을, 토큰은 안전한 비밀 저장 방식을 사용합니다.

## 실습

1. 사용자 CRUD Router와 컨트롤러 책임을 나누세요.
2. 생성 API의 성공·실패 테스트 케이스를 세 개 작성하세요.
3. API 클라이언트 테스트와 자동 테스트의 역할 차이를 설명하세요.

<details>
<summary>답</summary>

생성 성공은 유효한 본문에 201과 ID를 확인하고, 이름 누락은 400과 오류 코드를, 저장소 실패는 내부 정보 없는 500을 확인합니다. API 클라이언트는 탐색과 공유에 유리하고 자동 테스트는 반복 가능한 회귀 검증에 유리합니다.

</details>

## 더 알아보기

- [오류 처리와 404](05-error-handling-and-not-found.md)
- [REST API와 JSON 설계](06-rest-api-and-json-design.md)

## 체크리스트

- [ ] Router와 Controller 책임을 구분한다.
- [ ] 비즈니스 규칙을 HTTP에서 분리한다.
- [ ] CRUD 메서드와 상태를 일관되게 적용한다.
- [ ] 성공과 주요 실패 경로를 테스트한다.
- [ ] 탐색 테스트를 자동 회귀 테스트로 발전시킨다.

## 복습 질문 및 답변

### Q1. JSON API에는 View가 없나요?

<details>
<summary>답</summary>

템플릿 화면은 없을 수 있지만 클라이언트에 보낼 JSON 표현을 구성하는 계층이 View 역할을 한다고 볼 수 있습니다.

</details>

### Q2. Controller에서 데이터베이스 쿼리를 직접 작성하면 안 되나요?

<details>
<summary>답</summary>

작은 예제에서는 가능하지만 규모가 커지면 HTTP와 저장소 책임이 결합됩니다. 서비스·저장소 계층으로 분리하면 규칙 재사용과 테스트가 쉬워집니다.

</details>

### Q3. Postman 테스트만 있으면 자동 테스트가 필요 없나요?

<details>
<summary>답</summary>

아닙니다. 수동 탐색은 이해와 문서화에 좋지만 반복 가능한 회귀 검증과 CI 품질 게이트에는 자동 테스트가 필요합니다.

</details>

## 요약

MVC 관점은 Router, Controller와 데이터 책임을 분리해 CRUD API의 변경 범위를 줄입니다. 성공과 실패 계약을 자동 테스트로 고정하면 리팩터링에도 안정성을 유지할 수 있습니다.
