# Node.js 런타임과 실행 구조

Node.js는 브라우저 밖에서도 JavaScript를 실행하게 해 서버, 도구, 자동화 프로그램을 같은 언어로 만들 수 있게 합니다.

**핵심 키워드:** 런타임, V8, 서버, 싱글 스레드, LTS

## 핵심 내용

- Node.js는 JavaScript 엔진과 운영체제 기능을 연결한 실행 환경입니다.
- 브라우저 DOM 대신 파일, 프로세스, 네트워크 같은 서버 측 API를 제공합니다.
- JavaScript 코드는 기본적으로 한 메인 스레드에서 실행되지만 모든 작업이 한 스레드에서 처리되는 것은 아닙니다.
- 느린 입출력은 운영체제와 내부 작업 풀에 맡기고 완료 알림을 받아 처리합니다.
- 프로젝트에서는 최신 버전보다 지원 기간과 생태계 호환성이 안정적인 LTS를 우선 검토합니다.

## Node.js로 할 수 있는 일

웹 API와 실시간 서버, 명령줄 도구, 빌드 도구, 배치 작업과 자동화를 만들 수 있습니다. 브라우저와 서버가 같은 JavaScript 문법을 사용해도 실행 환경과 제공 API는 다르므로 코드를 그대로 옮길 수 있는지는 별도로 확인해야 합니다.

| 구분 | 브라우저 JavaScript | Node.js |
|---|---|---|
| 주요 목적 | 화면과 사용자 상호작용 | 서버·도구·자동화 실행 |
| 대표 전역 | `window`, `document` | `process`, `Buffer` |
| 파일 접근 | 기본적으로 제한 | 권한 범위에서 가능 |
| 모듈 | ES Module 중심 | CommonJS와 ES Module 지원 |

## 첫 프로그램 실행

```javascript
// app.js
const message = "Hello, Node.js";
console.log(message);
console.log(process.version);
```

```bash
node app.js
```

- **목적:** 파일을 Node.js 런타임으로 실행하고 환경 정보를 확인합니다.
- **흐름:** 소스 로드 → JavaScript 평가 → 표준 출력 → 프로세스 종료입니다.
- **결과:** 메시지와 실행 중인 Node.js 버전이 출력됩니다.
- **실무 포인트:** 팀에서는 버전 관리 파일과 CI 설정으로 런타임 버전을 맞춰 환경 차이를 줄입니다.

## 싱글 스레드의 의미

JavaScript 콜 스택은 한 번에 하나의 코드를 실행하므로 공유 상태를 다루기 단순합니다. 반면 긴 CPU 계산이 메인 스레드를 점유하면 다른 콜백도 실행되지 못합니다. 입출력 중심 서비스에는 효율적이지만 CPU 집약 작업은 작업 분리, Worker Threads 또는 별도 서비스 등을 검토합니다.

## 실습

1. 현재 Node.js 버전과 실행 플랫폼을 출력하세요.
2. 브라우저 전용 API와 Node.js 전용 API를 각각 두 개 적으세요.
3. 긴 동기 계산이 서버 응답을 막는 이유를 설명하세요.

<details>
<summary>답</summary>

```javascript
console.log(process.version);
console.log(process.platform);
```

브라우저에는 `window`, `document`, Node.js에는 `process`, `Buffer`가 대표적입니다. 긴 동기 계산은 메인 JavaScript 콜 스택을 점유해 다른 콜백의 실행을 지연시킵니다.

</details>

## 더 알아보기

- [비동기 프로그래밍 패턴](02-asynchronous-programming-patterns.md)
- [이벤트 루프와 작업 큐](03-event-loop-and-task-queues.md)

## 체크리스트

- [ ] Node.js가 런타임임을 설명할 수 있다.
- [ ] 브라우저와 Node.js API를 구분한다.
- [ ] 싱글 스레드와 비동기 처리를 혼동하지 않는다.
- [ ] CPU 작업이 이벤트 루프에 미치는 영향을 안다.
- [ ] 프로젝트에 적합한 LTS 버전을 선택한다.

## 복습 질문 및 답변

### Q1. Node.js는 프로그래밍 언어인가요?

<details>
<summary>답</summary>

아닙니다. JavaScript를 브라우저 밖에서 실행하고 운영체제 기능을 사용할 수 있게 하는 런타임입니다.

</details>

### Q2. 싱글 스레드라면 파일 읽기도 메인 스레드가 끝날 때까지 처리하나요?

<details>
<summary>답</summary>

비동기 파일 읽기는 운영체제나 내부 작업 풀에 위임될 수 있습니다. 완료 콜백의 JavaScript 실행은 이벤트 루프를 통해 메인 스레드에서 처리됩니다.

</details>

### Q3. 항상 가장 최신 Node.js 버전을 써야 하나요?

<details>
<summary>답</summary>

항상 그렇지는 않습니다. 운영 프로젝트에서는 지원 기간, 라이브러리 호환성, 배포 환경을 고려해 적절한 LTS를 선택하는 것이 일반적입니다.

</details>

## 요약

Node.js는 JavaScript 엔진과 서버 측 API를 제공하는 런타임입니다. 한 메인 콜 스택과 비동기 입출력 구조의 장단점을 이해해야 안정적인 서버 코드를 설계할 수 있습니다.
