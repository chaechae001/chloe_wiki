# Promise 체이닝과 async/await

비동기 단계가 여러 개라면 각 단계가 다음 단계에 값을 정확히 넘겨야 합니다. Promise 체이닝은 반환값을 연결하고, `async`/`await`는 같은 흐름을 위에서 아래로 읽히게 표현합니다.

## 핵심 키워드

Promise 체이닝 · 반환값 · 오류 전파 · async · await · try/catch

## 핵심 요약

- `then`, `catch`, `finally`는 항상 새 Promise를 반환합니다.
- `then`에서 일반 값을 반환하면 다음 `then`의 성공값이 됩니다.
- Promise를 반환하면 다음 단계는 그 Promise의 정착을 기다립니다.
- `async` 함수는 Promise를 반환하며, `await`는 해당 함수의 나머지 실행을 미룹니다.

## 1. 체인의 핵심은 return

```javascript
function loadUserId() {
  return Promise.resolve(17);
}

loadUserId()
  .then((id) => {
    return { id, role: "editor" };
  })
  .then((user) => {
    console.log(user.role);
  });
```

첫 `then`의 반환값이 새 Promise의 성공값이 되어 다음 `then`으로 전달됩니다. 반환을 빠뜨리면 다음 단계는 `undefined`를 받으며, 비동기 작업의 완료도 기다리지 않습니다.

> 한 줄 정리: 체인에서 다음 단계가 알아야 할 값이나 Promise는 반드시 `return`합니다.

## 2. 오류는 가장 가까운 처리기로 전파된다

```javascript
Promise.resolve("설정")
  .then(() => {
    throw new Error("형식 오류");
  })
  .then(() => {
    console.log("이 코드는 건너뜁니다.");
  })
  .catch((error) => {
    console.error(error.message);
    return "기본 설정";
  })
  .then((config) => console.log(config));
```

`then` 콜백에서 던진 오류는 반환된 Promise를 rejected로 만듭니다. `catch`가 정상 값을 반환하면 체인은 다시 성공 흐름으로 복구될 수 있습니다.

## 3. async/await로 같은 흐름 표현하기

```javascript
async function prepareConfig() {
  try {
    const name = await Promise.resolve("dark");
    return { theme: name };
  } catch (error) {
    return { theme: "system" };
  }
}

prepareConfig().then(console.log);
```

`await` 오른쪽 Promise가 성공하면 그 값을 반환하고, 실패하면 예외를 던집니다. 그래서 `try/catch/finally`로 성공·실패·정리를 표현할 수 있습니다.

### 자주 헷갈리는 점

- `await`는 `async` 함수 안에서 사용하는 것이 기본입니다.
- `async` 함수가 일반 값을 반환해도 호출자는 Promise를 받습니다.
- 서로 독립적인 작업을 차례로 `await`하면 불필요하게 직렬 실행될 수 있습니다.

## 대표 코드: 순차 의존 작업

### 목적

사용자 ID가 있어야 설정을 읽을 수 있는 의존 관계를 Promise 체인과 `async` 함수로 표현합니다.

```javascript
function findUser() {
  return Promise.resolve({ id: 42, name: "Mina" });
}

function findPreferences(userId) {
  return Promise.resolve({ userId, language: "ko" });
}

async function buildProfile() {
  const user = await findUser();
  const preferences = await findPreferences(user.id);

  return { ...user, ...preferences };
}
```

### 흐름

1. `findUser`의 Promise가 정착할 때까지 함수의 나머지를 미룹니다.
2. 얻은 `user.id`로 두 번째 작업을 시작합니다.
3. 두 결과를 합쳐 반환하면 fulfilled Promise가 됩니다.

### 결과와 실무 활용

인증 후 데이터 조회, 생성 후 상세 조회처럼 앞선 결과가 다음 입력인 작업에 적합합니다. 각 단계가 독립적이라면 `Promise.all`로 함께 시작하는 편이 효율적입니다.

## 연습 문제

아래 체인에서 두 번째 `then`이 숫자 `12`를 받도록 빈칸을 채워 보세요.

```javascript
Promise.resolve(5)
  .then((value) => {
    // 빈칸
  })
  .then((result) => console.log(result));
```

<details>
<summary>답</summary>

```javascript
Promise.resolve(5)
  .then((value) => {
    return value + 7;
  })
  .then((result) => console.log(result));
```

</details>

## 비교: then 체인 vs async/await

| 구분 | then 체인 | async/await |
|---|---|---|
| 읽는 방식 | 변환 단계를 연결 | 위에서 아래로 서술 |
| 오류 처리 | `catch` | `try/catch` |
| 조건·반복 | 체인이 복잡해질 수 있음 | 일반 제어문과 자연스럽게 결합 |
| 공통 기반 | Promise | Promise |

## 연결되는 개념

- [Promise 상태와 후속 처리](04-promise-states-and-handlers.md)
- 독립 작업의 병렬 시작은 [마이크로태스크와 Promise 조합](06-microtasks-and-promise-combinators.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] `then`에서 반환한 값의 이동 경로를 설명한다.
- [ ] 체인에서 오류가 `catch`까지 전파되는 방식을 안다.
- [ ] 순차 의존 작업과 독립 작업을 구분한다.

## 복습 질문 및 답변

### Q1. `then`에서 Promise를 반환하면 다음 단계는 어떻게 되는가?

<details>
<summary>답</summary>

다음 `then`은 반환된 Promise가 정착할 때까지 기다리고, 그 성공값 또는 실패 이유를 이어받습니다.

</details>

### Q2. `async` 함수에서 예외를 던지면 호출자는 무엇을 받는가?

<details>
<summary>답</summary>

그 예외를 실패 이유로 가진 rejected Promise를 받습니다.

</details>

### Q3. 서로 독립적인 두 작업을 연속으로 await할 때의 단점은?

<details>
<summary>답</summary>

첫 작업이 끝난 뒤 두 번째를 시작하므로 전체 시간이 불필요하게 길어질 수 있습니다. 함께 시작한 뒤 `Promise.all`로 기다리는 방법을 고려합니다.

</details>

> 최종 한 줄: Promise 체인의 `return`과 async/await의 오류 흐름을 이해하면 여러 비동기 단계를 예측 가능하게 연결할 수 있습니다.
