# 마이크로태스크와 Promise 조합

타이머와 Promise가 함께 있으면 “둘 다 비동기”라는 설명만으로 출력 순서를 알 수 없습니다. Promise 후속 콜백은 마이크로태스크 큐를 사용하며, 일반 태스크보다 먼저 처리됩니다.

## 핵심 키워드

마이크로태스크 큐 · 태스크 큐 · 우선순위 · Promise.resolve · Promise.reject · Promise.all

## 핵심 요약

- Promise의 `then`, `catch`, `finally` 콜백은 마이크로태스크로 예약됩니다.
- 현재 동기 코드가 끝나면 다음 태스크 전에 마이크로태스크 큐를 모두 비웁니다.
- `Promise.resolve`와 `Promise.reject`는 이미 정착된 Promise를 간단히 만듭니다.
- `Promise.all`은 모든 작업의 성공 결과를 입력 순서대로 모으고, 하나라도 실패하면 거부됩니다.

## 1. 마이크로태스크 우선순위

```javascript
console.log("sync-start");

setTimeout(() => console.log("task"), 0);

Promise.resolve().then(() => {
  console.log("microtask");
});

console.log("sync-end");
```

일반적인 결과는 `sync-start → sync-end → microtask → task`입니다. 동기 코드가 끝난 뒤 마이크로태스크 큐를 먼저 비우고, 그 다음 타이머 태스크를 실행합니다.

> 한 줄 정리: 현재 태스크가 끝나면 Promise 후속 작업을 먼저 처리한 뒤 다음 타이머나 이벤트 태스크로 넘어갑니다.

## 2. 마이크로태스크가 다시 마이크로태스크를 만들 때

```javascript
Promise.resolve()
  .then(() => {
    console.log("first");
    Promise.resolve().then(() => console.log("nested"));
  })
  .then(() => console.log("second"));
```

마이크로태스크 실행 중 새 마이크로태스크가 추가될 수 있습니다. 큐에 추가된 순서에 따라 `first → nested → second`가 됩니다. 마이크로태스크를 끝없이 생성하면 다음 태스크와 렌더링이 지연될 수 있습니다.

## 3. 여러 Promise를 함께 기다리기

```javascript
const profilePromise = Promise.resolve({ name: "Mina" });
const themePromise = Promise.resolve("dark");

Promise.all([profilePromise, themePromise])
  .then(([profile, theme]) => {
    console.log(profile.name, theme);
  });
```

`Promise.all`은 인자로 받은 작업을 차례로 실행하는 명령이 아닙니다. 배열을 만들기 전에 각 Promise가 이미 시작될 수 있으며, 모두 완료될 때까지 기다렸다가 입력 순서대로 결과를 제공합니다.

### 실패 전략

```javascript
Promise.all([
  Promise.resolve("primary"),
  Promise.reject(new Error("backup failed")),
]).catch((error) => console.error(error.message));
```

하나라도 실패하면 반환 Promise는 rejected가 됩니다. 다른 작업이 자동으로 취소되는 것은 아니므로 취소가 필요하면 별도의 API를 설계해야 합니다.

### 자주 헷갈리는 점

- 완료 순서가 달라도 `Promise.all` 결과 배열은 입력 순서를 유지합니다.
- `Promise.resolve(value)`는 비동기 작업을 새로 시작한다는 뜻이 아니라 값을 Promise 인터페이스로 감쌉니다.
- 마이크로태스크 우선은 동기 코드보다 먼저라는 뜻이 아닙니다. 현재 동기 코드는 항상 먼저 끝납니다.

## 대표 코드: 독립 데이터 병렬 준비

### 목적

서로 의존하지 않는 설정과 권한 정보를 함께 시작해 전체 대기 시간을 줄입니다.

```javascript
function loadSettings() {
  return Promise.resolve({ locale: "ko" });
}

function loadPermissions() {
  return Promise.resolve(["read", "write"]);
}

async function initializePage() {
  const [settings, permissions] = await Promise.all([
    loadSettings(),
    loadPermissions(),
  ]);

  return { settings, permissions };
}
```

### 흐름

1. 두 함수가 연속으로 호출되어 Promise를 만듭니다.
2. `Promise.all`이 두 결과를 함께 기다립니다.
3. 모두 성공하면 입력 배열과 같은 위치의 결과를 구조 분해합니다.

### 결과와 실무 활용

페이지 초기화에 필요한 독립 API 요청, 여러 파일의 메타데이터 조회처럼 함께 시작할 수 있는 작업에 유용합니다. 일부 실패를 허용해야 한다면 각 작업에서 오류를 복구하거나 다른 조합 메서드를 검토합니다.

## 연습 문제

다음 코드의 출력 순서를 적어 보세요.

```javascript
setTimeout(() => console.log("A"), 0);
Promise.resolve().then(() => console.log("B"));
console.log("C");
```

<details>
<summary>답</summary>

`C → B → A`입니다. 동기 코드 `C`가 먼저 실행되고, 마이크로태스크인 Promise 콜백 `B`, 태스크인 타이머 콜백 `A` 순서로 이어집니다.

</details>

## 비교: 순차 await vs Promise.all

| 구분 | 순차 await | Promise.all |
|---|---|---|
| 시작 시점 | 앞 작업 후 다음 작업 | 여러 작업을 함께 시작 가능 |
| 적합한 관계 | 앞 결과에 의존 | 서로 독립 |
| 실패 | 해당 지점에서 예외 | 하나라도 실패하면 전체 거부 |
| 결과 | 단계별 값 | 입력 순서의 배열 |

## 연결되는 개념

- [태스크 큐와 실행 순서](03-task-queue-and-execution-order.md)
- [Promise 상태와 후속 처리](04-promise-states-and-handlers.md)
- [Promise 체이닝과 async/await](05-promise-chaining-and-async-await.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 동기 코드, 마이크로태스크, 태스크의 순서를 예측한다.
- [ ] `Promise.all` 결과 배열의 순서 규칙을 안다.
- [ ] 의존 작업과 독립 작업에 맞는 실행 방식을 선택한다.

## 복습 질문 및 답변

### Q1. Promise 콜백은 항상 타이머 콜백보다 먼저 실행되는가?

<details>
<summary>답</summary>

같은 현재 태스크에서 둘 다 예약되고 Promise가 정착되어 있다면 마이크로태스크가 다음 태스크보다 먼저 처리됩니다. 하지만 서로 다른 시점과 태스크에서 등록된 모든 상황을 단순히 “항상”으로 일반화하면 안 됩니다.

</details>

### Q2. `Promise.all`의 결과 순서는 완료 순서인가?

<details>
<summary>답</summary>

아닙니다. 각 Promise의 완료 순서와 관계없이 입력 배열의 순서를 유지합니다.

</details>

### Q3. `Promise.all`에서 하나가 실패하면 다른 작업도 취소되는가?

<details>
<summary>답</summary>

아닙니다. 반환된 Promise가 즉시 거부될 뿐, 이미 시작된 다른 작업은 자체적으로 계속될 수 있습니다.

</details>

> 최종 한 줄: Promise 후속 처리는 마이크로태스크로 우선 처리되고, `Promise.all`은 독립 작업의 결과를 효율적으로 모읍니다.
