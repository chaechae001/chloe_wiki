# 비동기 프로그래밍 패턴

네트워크와 파일 입출력을 기다리는 동안 다른 일을 처리하려면 결과가 나중에 도착한다는 사실을 코드 구조에 반영해야 합니다.

**핵심 키워드:** Callback, Promise, async, await, 병렬 실행

## 핵심 내용

- 콜백은 완료 후 실행할 함수를 전달하는 가장 직접적인 비동기 표현입니다.
- 중첩 콜백이 깊어지면 흐름과 오류 처리가 분산될 수 있습니다.
- Promise는 대기, 성공, 실패 상태와 후속 동작을 체인으로 표현합니다.
- `async`/`await`는 Promise 기반 코드를 순차적인 문장처럼 읽게 합니다.
- 서로 독립적인 작업은 `Promise.all`로 함께 시작할 수 있지만 하나의 실패가 전체 결과를 거부합니다.

## 콜백에서 Promise로

```javascript
import { readFile } from "node:fs/promises";

function loadConfig(path) {
  return readFile(path, "utf8").then(JSON.parse);
}
```

Promise를 반환하면 호출자는 성공과 실패를 하나의 체인에서 처리할 수 있습니다.

```javascript
loadConfig("config.json")
  .then((config) => console.log(config.mode))
  .catch((error) => console.error("설정 읽기 실패", error));
```

## async/await와 오류 처리

```javascript
async function printMode() {
  try {
    const text = await readFile("config.json", "utf8");
    const config = JSON.parse(text);
    console.log(config.mode);
  } catch (error) {
    console.error("설정 처리 실패", error);
  }
}
```

- **목적:** 비동기 파일 읽기와 파싱 실패를 한 범위에서 처리합니다.
- **흐름:** Promise 대기 → 성공 값 변환 → 사용, 또는 예외를 `catch`로 전달합니다.
- **결과:** 호출 흐름이 선형으로 보이고 실패 경로가 명확해집니다.
- **실무 포인트:** 오류를 출력만 하고 삼키기보다 복구하거나 상위 호출자에게 다시 전달할지 결정합니다.

## 순차 실행과 병렬 실행

| 방식 | 예시 | 적합한 경우 |
|---|---|---|
| 순차 | `await first(); await second();` | 두 번째가 첫 결과에 의존 |
| 병렬 | `await Promise.all([first(), second()])` | 서로 독립적이고 모두 필요 |

```javascript
const [user, products] = await Promise.all([
  fetchUser(),
  fetchProducts(),
]);
```

독립 작업을 순차로 기다리면 전체 시간이 불필요하게 늘어날 수 있습니다. 반대로 의존성이 있거나 호출량 제한이 있다면 무조건 병렬화해서는 안 됩니다.

## 실습

1. Promise를 반환하는 함수의 성공과 실패를 처리하세요.
2. 두 독립 조회를 `Promise.all`로 병렬 실행하세요.
3. `async` 함수에서 오류를 복구할 경우와 다시 던질 경우를 구분하세요.

<details>
<summary>답</summary>

```javascript
async function loadPage() {
  try {
    const [profile, posts] = await Promise.all([fetchProfile(), fetchPosts()]);
    return { profile, posts };
  } catch (error) {
    throw new Error("페이지 데이터 로드 실패", { cause: error });
  }
}
```

</details>

## 더 알아보기

- [Node.js 런타임과 실행 구조](01-nodejs-runtime-and-architecture.md)
- [이벤트 루프와 작업 큐](03-event-loop-and-task-queues.md)

## 체크리스트

- [ ] 콜백, Promise, async/await의 관계를 안다.
- [ ] 비동기 함수가 Promise를 반환함을 이해한다.
- [ ] 오류의 복구와 전파를 구분한다.
- [ ] 의존 작업은 순차로 처리한다.
- [ ] 독립 작업만 적절히 병렬화한다.

## 복습 질문 및 답변

### Q1. `await`를 쓰면 비동기 작업이 동기로 바뀌나요?

<details>
<summary>답</summary>

아닙니다. 현재 async 함수의 진행을 잠시 멈춰 보이게 할 뿐, Promise가 처리되는 동안 런타임은 다른 이벤트를 처리할 수 있습니다.

</details>

### Q2. `Promise.all` 중 하나가 실패하면 어떻게 되나요?

<details>
<summary>답</summary>

반환된 Promise는 해당 실패 이유로 거부됩니다. 이미 시작한 다른 작업이 자동으로 취소되는 것은 아닙니다.

</details>

### Q3. 콜백 자체가 나쁜 방식인가요?

<details>
<summary>답</summary>

아닙니다. 이벤트 처리 등에서 여전히 핵심입니다. 다만 의존 작업을 깊게 중첩하면 가독성과 오류 관리가 어려워질 수 있습니다.

</details>

## 요약

비동기 패턴은 나중에 도착하는 결과와 실패를 표현하는 방법입니다. Promise와 async/await로 흐름을 명확히 하고, 의존성과 실패 정책을 고려해 병렬 실행을 선택합니다.
