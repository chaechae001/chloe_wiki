# 순차 실행과 병렬 실행

`await`를 어디에 놓느냐에 따라 전체 대기 시간이 달라집니다. 중요한 기준은 속도가 아니라 작업 사이의 의존 관계입니다.

## 핵심 키워드

순차 실행 · 병렬 시작 · 의존 관계 · `Promise.all` · 실패 전략

## 핵심 요약

- 앞 결과가 다음 작업의 입력이면 순차 실행해야 합니다.
- 서로 독립적인 작업은 먼저 시작하고 함께 기다릴 수 있습니다.
- 연속된 `await`는 각 작업을 앞 작업 완료 후 시작하게 만들 수 있습니다.
- `Promise.all`은 입력 순서대로 결과를 제공하지만 하나라도 실패하면 전체가 거부됩니다.

## 1. 의존 작업의 순차 실행

### 정의와 필요성

사용자 정보를 찾아야 그 사용자의 ID로 상세 설정을 요청할 수 있는 것처럼, 앞 단계의 결과가 다음 단계에 필요하면 실행 순서를 바꿀 수 없습니다.

```javascript
function findAccount(username) {
  return Promise.resolve({ id: 21, username });
}

function findPreferences(accountId) {
  return Promise.resolve({ accountId, locale: "ko" });
}

async function loadProfile(username) {
  const account = await findAccount(username);
  const preferences = await findPreferences(account.id);

  return { account, preferences };
}
```

### 흐름

```text
사용자 이름
    ↓
계정 조회 완료
    ↓ account.id
환경 설정 조회
    ↓
두 결과 조합
```

두 번째 호출은 `account.id` 없이는 시작할 수 없으므로 순차 실행이 올바릅니다.

> 한 줄 정리: 데이터 의존성이 실행 순서를 결정합니다.

## 2. 독립 작업의 병렬 시작

### 정의와 필요성

화면에 필요한 공지와 테마 설정이 서로 영향을 주지 않는다면 첫 작업이 끝날 때까지 두 번째 시작을 미룰 이유가 없습니다.

```javascript
function loadNotice() {
  return Promise.resolve("점검 안내");
}

function loadTheme() {
  return Promise.resolve("light");
}

async function initializeScreen() {
  const [notice, theme] = await Promise.all([
    loadNotice(),
    loadTheme(),
  ]);

  return { notice, theme };
}
```

배열 안의 함수들이 먼저 호출되어 작업을 시작하고, `Promise.all`이 두 결과를 함께 기다립니다. 결과 배열은 완료 순서가 아니라 입력 순서를 유지합니다.

### 순차 await와 시간 차이

```javascript
async function compareLoadingStrategies() {
  // 독립 작업을 불필요하게 순차 실행
  const first = await loadNotice();
  const second = await loadTheme();

  // 독립 작업을 함께 시작
  const [notice, theme] = await Promise.all([
    loadNotice(),
    loadTheme(),
  ]);

  return { first, second, notice, theme };
}
```

각 작업이 1초라면 첫 방식은 약 2초, 두 번째는 약 1초가 될 수 있습니다. 실제 시간은 환경과 작업 특성에 따라 달라집니다.

### 자주 헷갈리는 점

- `Promise.all`이 CPU 코드를 별도 스레드에서 병렬 실행하는 것은 아닙니다.
- 하나가 실패해도 이미 시작된 나머지 작업이 자동으로 취소되지는 않습니다.
- 성공 결과 배열은 작업 완료 순서가 아닌 입력 순서입니다.

## 대표 코드: 대시보드 초기화

### 목적

필수 사용자 정보는 먼저 확인하고, 이후 서로 독립적인 두 데이터를 함께 불러옵니다.

```javascript
function loadSession() {
  return Promise.resolve({ userId: 8 });
}

function loadTasks(userId) {
  return Promise.resolve([{ id: 1, owner: userId }]);
}

function loadMessages(userId) {
  return Promise.resolve([{ id: 2, receiver: userId }]);
}

async function initializeDashboard() {
  const session = await loadSession();

  const [tasks, messages] = await Promise.all([
    loadTasks(session.userId),
    loadMessages(session.userId),
  ]);

  return { session, tasks, messages };
}
```

### 흐름과 실무 활용

세션은 두 조회의 공통 입력이라 먼저 기다립니다. 세션 이후의 두 요청은 독립적이므로 함께 시작합니다. 인증 후 여러 위젯을 불러오는 화면에서 자주 사용하는 혼합 구조입니다.

## 직접 해보기

서로 독립적인 `loadMenu()`와 `loadBanner()`를 함께 실행하고 `{ menu, banner }`를 반환하는 async 함수를 작성하세요.

<details>
<summary>답</summary>

```javascript
async function loadHome() {
  const [menu, banner] = await Promise.all([
    loadMenu(),
    loadBanner(),
  ]);

  return { menu, banner };
}
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| 순차 await vs Promise.all | 앞 결과가 필요하면 순차 await, 독립 작업이면 함께 시작을 고려합니다. |
| 병렬 시작 vs 동시 코드 실행 | 대기 시간이 겹치는 것이며 자바스크립트 실행 스레드가 늘어나는 것은 아닙니다. |
| 실패 반환 vs 작업 취소 | Promise.all의 결과는 빠르게 거부되지만 다른 작업이 자동 취소되지는 않습니다. |

## 연결되는 개념

- [async 함수와 await의 기본](01-async-functions-and-await.md)
- 여러 항목 처리 전략은 [비동기 반복과 작업 흐름 설계](06-async-iteration-and-workflow-design.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 작업 사이의 데이터 의존성을 찾을 수 있다.
- [ ] 독립 작업을 함께 시작할 수 있다.
- [ ] Promise.all의 결과와 실패 규칙을 설명할 수 있다.

## 복습 질문 및 답변

### Q1. 앞 API 결과의 ID가 다음 API에 필요할 때 Promise.all을 써도 되는가?

<details>
<summary>답</summary>

두 번째 호출을 시작할 입력이 아직 없으므로 해당 두 단계는 순차 실행해야 합니다.

</details>

### Q2. Promise.all 결과 배열은 완료가 빠른 순서인가?

<details>
<summary>답</summary>

아닙니다. 전달한 Promise 배열의 입력 순서를 유지합니다.

</details>

### Q3. 독립 작업을 연속 await하면 어떤 문제가 생길 수 있는가?

<details>
<summary>답</summary>

첫 작업의 대기 시간이 끝난 뒤 두 번째를 시작하므로 전체 대기 시간이 불필요하게 길어질 수 있습니다.

</details>

> 최종 한 줄: 순차와 병렬의 선택 기준은 문법이 아니라 작업 간 의존 관계입니다.
