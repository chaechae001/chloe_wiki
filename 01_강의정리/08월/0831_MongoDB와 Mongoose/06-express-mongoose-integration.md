# Express와 Mongoose 연결 관리

Express 서버는 데이터베이스 연결 준비와 실패를 명확히 관리해야 요청을 받았지만 처리할 수 없는 상태를 피할 수 있습니다.

**핵심 키워드:** connection, lifecycle, environment variable, graceful shutdown, error handling

## 핵심 내용

- 연결 문자열은 환경변수나 비밀 관리 시스템에서 주입합니다.
- 초기 연결이 성공한 뒤 서버가 요청을 받도록 시작 순서를 구성합니다.
- 연결 실패를 로그만 남기고 계속 실행하지 말고 프로세스 정책에 따라 실패시킵니다.
- 연결 이벤트는 관측에 활용하되 같은 이벤트 리스너를 반복 등록하지 않습니다.
- 종료 신호에서 새 요청을 막고 서버와 데이터베이스 연결을 순서대로 닫습니다.

## 시작 수명주기

```javascript
import express from "express";
import mongoose from "mongoose";

async function start() {
  const uri = process.env.MONGODB_URI;
  if (!uri) throw new Error("MONGODB_URI is required");

  await mongoose.connect(uri);

  const app = express();
  app.use(express.json());
  return app.listen(process.env.PORT ?? 3000);
}

start().catch((error) => {
  console.error("startup failed", error);
  process.exitCode = 1;
});
```

- **목적:** 데이터베이스가 준비된 경우에만 HTTP 요청을 받습니다.
- **흐름:** 설정 확인 → DB 연결 → 앱 구성 → 포트 수신 → 실패 시 종료 상태 설정입니다.
- **결과:** 준비되지 않은 서버가 정상처럼 노출되는 시간을 줄입니다.
- **실무 포인트:** 실제 로그에는 연결 문자열 전체를 출력하지 않습니다.

## 코드 위치

| 위치 | 책임 |
|---|---|
| config | 환경변수 읽기와 형식 검증 |
| database | 연결, 이벤트와 종료 |
| model | Schema와 Model 정의 |
| service/repository | Model을 이용한 데이터 작업 |
| controller | HTTP 입력과 응답 변환 |

Model 파일마다 connect를 호출하지 않고 애플리케이션 시작 단계에서 연결을 한 번 관리합니다. 테스트는 별도 데이터베이스를 사용하고 각 테스트의 데이터 격리 전략을 둡니다.

## 연결 이벤트와 상태

```javascript
mongoose.connection.on("connected", () => console.log("database connected"));
mongoose.connection.on("disconnected", () => console.warn("database disconnected"));
mongoose.connection.on("error", (error) => console.error("database error", error));
```

이벤트 로그는 원인을 파악하는 단서지만 자동 재시도와 장애 대응을 대신하지 않습니다. 드라이버의 재연결 동작, 애플리케이션 readiness, 알림 정책을 함께 설계합니다.

## 안전한 종료

종료 신호를 받으면 HTTP 서버가 새 연결을 받지 않도록 닫고, 진행 중 요청에 시간을 준 뒤 Mongoose 연결을 종료합니다. 무한 대기를 피하기 위한 제한 시간도 필요합니다.

## 실습

1. 환경변수가 없으면 시작을 중단하는 코드를 작성하세요.
2. DB 연결 전후 서버 시작 순서를 설명하세요.
3. 안전한 종료 단계와 제한 시간이 필요한 이유를 적으세요.

<details>
<summary>답</summary>

설정을 검증하고 `await mongoose.connect(uri)`가 성공한 뒤 `app.listen`을 호출합니다. 종료 시 새 요청 중단 → 진행 요청 정리 → DB 연결 종료 순서를 사용하며, 제한 시간은 종료가 영원히 멈추는 상황을 방지합니다.

</details>

## 더 알아보기

- [MongoDB 핵심 구조와 운영 방식](02-mongodb-core-concepts-and-operations.md)
- [ODM과 ORM 선택](07-odm-and-orm-selection.md)

## 체크리스트

- [ ] 연결 문자열을 비밀로 관리한다.
- [ ] 연결 성공 후 서버를 시작한다.
- [ ] 초기 연결 실패 정책을 정의한다.
- [ ] 연결 이벤트를 관측한다.
- [ ] 안전한 종료와 제한 시간을 구현한다.

## 복습 질문 및 답변

### Q1. 요청이 들어올 때마다 `mongoose.connect`를 호출해야 하나요?

<details>
<summary>답</summary>

일반적인 장기 실행 서버에서는 시작 단계에서 연결과 풀을 구성해 재사용합니다. 실행 환경의 수명주기에 맞는 연결 전략이 필요합니다.

</details>

### Q2. 연결 문자열을 로그로 출력하면 왜 위험한가요?

<details>
<summary>답</summary>

사용자명, 비밀번호와 호스트 정보가 포함될 수 있어 로그 접근자에게 데이터베이스 권한이 노출될 수 있습니다.

</details>

### Q3. 서버가 포트를 열었으면 readiness가 보장되나요?

<details>
<summary>답</summary>

아닙니다. 필수 데이터베이스나 외부 의존성이 준비되지 않았다면 요청을 정상 처리할 수 없으므로 별도의 준비 상태 판단이 필요합니다.

</details>

## 요약

Express와 Mongoose 통합의 핵심은 CRUD 코드보다 연결 수명주기입니다. 비밀 설정, 준비 순서, 장애 관측과 안전한 종료를 하나의 운영 흐름으로 설계하세요.
