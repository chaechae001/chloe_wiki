# 이벤트 루프와 Promise

> JavaScript는 한 번에 하나의 코드를 실행하지만, 기다림까지 모두 멈춰서 처리할 필요는 없습니다.

`비동기` · `Call Stack` · `Task Queue` · `Event Loop` · `Promise`

## 핵심요약

- 긴 동기 작업은 메인 스레드와 화면 반응을 함께 막습니다.
- 브라우저는 타이머와 네트워크 대기를 맡고 완료된 콜백을 큐에 보냅니다.
- 이벤트 루프는 Call Stack이 비었을 때 대기 작업의 실행을 연결합니다.
- Promise는 미래의 성공 값 또는 실패 이유를 표현합니다.
- `then`, `catch`, `finally`로 성공·실패·정리 흐름을 분리할 수 있습니다.

## 1. 동기 실행과 비동기 실행

동기 코드는 Call Stack에서 작성 순서대로 실행됩니다. 계산이 오래 걸리면 클릭 처리와 렌더링도 기다립니다. 비동기 처리는 네트워크나 타이머처럼 기다림이 있는 작업을 실행 환경에 맡기고, JavaScript가 다른 일을 계속하게 합니다.

```javascript
console.log('시작');
setTimeout(() => console.log('타이머 완료'), 0);
console.log('끝');
```

출력은 `시작 → 끝 → 타이머 완료`입니다. 지연 시간이 0이어도 콜백은 즉시 Stack에 끼어들지 않고 실행 가능한 시점까지 큐에서 기다립니다.

## 2. 이벤트 루프의 흐름

1. 동기 함수가 Call Stack에서 실행됩니다.
2. 타이머나 네트워크 대기는 브라우저 환경에 위임됩니다.
3. 작업이 끝나면 실행할 콜백이 큐에 들어갑니다.
4. 이벤트 루프가 빈 Call Stack을 확인합니다.
5. 대기 작업이 Stack으로 이동해 실행됩니다.

이 구조는 메인 스레드를 없애는 것이 아니라 ‘기다리는 시간’과 ‘JavaScript가 실행되는 시간’을 분리합니다.

## 3. Promise의 상태와 처리

Promise는 `pending`에서 시작해 한 번만 `fulfilled` 또는 `rejected`로 확정됩니다. 두 완료 상태를 합쳐 `settled`라고 부릅니다.

```javascript
function loadProfile(id) {
  return fetch(`/api/profiles/${id}`)
    .then((response) => {
      if (!response.ok) throw new Error('요청 실패');
      return response.json();
    });
}

loadProfile(7)
  .then((profile) => console.log(profile.name))
  .catch((error) => console.error(error.message))
  .finally(() => console.log('요청 종료'));
```

### 코드 목적

서버 응답을 JSON으로 바꾸고 성공 값, 오류, 종료 처리를 각각 분리합니다.

### 코드 흐름과 결과 해석

`fetch`가 Promise를 반환하고, 첫 `then`이 응답 상태를 검사합니다. 반환된 JSON Promise를 다음 핸들러가 이어받습니다. 중간에 던진 오류는 가까운 `catch`로 이동하고 `finally`는 결과와 관계없이 실행됩니다.

### 실무 연결

사용자 정보, 목록, 저장 요청처럼 완료 시점을 미리 알 수 없는 작업의 로딩·성공·실패 UI를 구성할 때 사용합니다.

## 직접 해보기

1. 위 타이머 예제의 출력 순서를 예측하세요.
2. `loadProfile`에서 HTTP 오류가 `catch`로 가는 이유를 설명하세요.
3. 버튼의 로딩 표시를 어느 시점에 켜고 끌지 설계하세요.

<details>
<summary>답</summary>

1. `시작`, `끝`, `타이머 완료` 순입니다.
2. `response.ok`가 거짓일 때 던진 Error가 Promise chain을 rejected 상태로 만들어 `catch`가 처리합니다.
3. 요청 직전에 로딩을 켜고 `finally`에서 끄면 성공과 실패 모두 처리할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| A vs B | 차이 |
|---|---|
| 동기 vs 비동기 | 실행을 끝까지 기다리면 동기, 완료 시점의 처리를 나누면 비동기 |
| pending vs settled | 처리 중인 상태 vs 성공 또는 실패로 확정된 상태 |
| `then` vs `finally` | 값을 이어 처리함 vs 결과와 무관한 정리를 수행함 |

## 연결되는 개념

- 다음에 이어지는 개념: [async/await와 Promise 조합](02-async-await-and-promise-combinators.md)
- 함께 보면 좋은 개념: [HTTP API와 CORS](03-http-api-openapi-and-cors.md)

## 셀프 체크

- [ ] Call Stack과 Task Queue의 역할을 설명할 수 있다.
- [ ] 0ms 타이머가 즉시 실행되지 않는 이유를 안다.
- [ ] Promise의 세 상태를 구분한다.
- [ ] `then`, `catch`, `finally`의 책임을 나눈다.
- [ ] 비동기 작업의 로딩과 오류 상태를 설계할 수 있다.

### 복습 질문 및 답변

**Q1. 이벤트 루프가 콜백을 실행 가능한 상태로 옮기는 조건은?**

<details>
<summary>답</summary>

현재 Call Stack이 비어 JavaScript가 새 작업을 실행할 수 있을 때입니다.

</details>

**Q2. Promise가 한 번 fulfilled된 뒤 rejected로 바뀔 수 있는가?**

<details>
<summary>답</summary>

아니요. Promise는 한 번 settled되면 상태와 결과가 다시 바뀌지 않습니다.

</details>

**Q3. 비동기 코드가 CPU 계산 자체를 빠르게 만드는가?**

<details>
<summary>답</summary>

아닙니다. 기다리는 동안 다른 일을 가능하게 할 뿐, 메인 스레드의 긴 계산은 여전히 UI를 막을 수 있습니다.

</details>

## 한 줄 정리

> 이벤트 루프는 기다림과 실행을 분리하고, Promise는 그 비동기 결과를 연결 가능한 값으로 표현합니다.
