# async 함수와 await의 기본

`async`/`await`는 Promise를 없애는 문법이 아닙니다. Promise의 생성과 정착 규칙은 그대로 두고, 비동기 작업의 순서를 일반적인 함수처럼 읽게 해 주는 표현 방식입니다.

## 핵심 키워드

`async` · `await` · Promise 반환 · 실행 중단과 재개 · rejected Promise

## 핵심 요약

- `async` 함수는 항상 Promise를 반환합니다.
- 일반 반환값은 fulfilled Promise의 성공값이 됩니다.
- 함수에서 던진 예외는 rejected Promise의 실패 이유가 됩니다.
- `await`는 성공값을 꺼내고 현재 함수의 나머지 실행을 나중으로 미룹니다.

## 1. async 함수의 반환 규칙

### 정의와 필요성

`async` 함수는 반환 형태를 Promise로 통일합니다. 호출자는 함수 내부가 즉시 값을 만들든 실제 비동기 작업을 수행하든 같은 방식으로 결과를 받을 수 있습니다.

```javascript
async function getTheme() {
  return "dark";
}

const result = getTheme();
console.log(result instanceof Promise); // true
result.then((theme) => console.log(theme)); // dark
```

`return "dark"`는 개념적으로 `return Promise.resolve("dark")`와 같습니다. 반대로 함수 내부에서 오류를 던지면 반환 Promise가 거부됩니다.

```javascript
async function loadRequiredValue(value) {
  if (value == null) {
    throw new Error("값이 필요합니다.");
  }

  return value;
}
```

### 자주 헷갈리는 점

- `async`가 함수 내부 코드를 자동으로 병렬 실행하지는 않습니다.
- async 함수의 직접 반환값은 일반 값이 아니라 Promise입니다.
- Promise를 반환하면 새 Promise로 이중 포장하지 않고 그 결과를 따라갑니다.

> 한 줄 정리: `async`는 함수의 성공값과 실패를 Promise 인터페이스로 통일합니다.

## 2. await의 실행 흐름

### 정의와 흐름

`await expression`은 표현식을 Promise로 해석하고 정착을 기다립니다. 기다리는 동안 호출 스택을 계속 점유하지 않으며, 해당 async 함수의 나머지 부분이 후속 작업으로 예약됩니다.

```javascript
function delay(ms, value) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(value), ms);
  });
}

async function showStatus() {
  console.log("요청 시작");
  const status = await delay(100, "준비 완료");
  console.log(status);
}

showStatus();
console.log("다른 작업");
```

출력 순서는 `요청 시작 → 다른 작업 → 준비 완료`입니다. `await`는 `showStatus`의 뒷부분만 미루며 바깥 코드는 계속 실행됩니다.

### 일반 값에 await를 붙이면

```javascript
async function normalize() {
  const value = await 7;
  return value * 2;
}
```

일반 값도 `Promise.resolve(7)`처럼 처리되므로 결과는 14입니다. 다만 실행 재개는 비동기 경계 뒤에서 일어나므로 불필요한 `await`는 쓰지 않는 편이 명확합니다.

## 대표 코드: 설정 로드 함수

### 목적

Promise 기반 저장소에서 설정을 읽고, 읽은 값을 일반 객체처럼 가공합니다.

```javascript
function readSetting(key) {
  return Promise.resolve({ key, value: "compact" });
}

async function buildViewOptions() {
  const setting = await readSetting("layout");

  return {
    layout: setting.value,
    animation: true,
  };
}

buildViewOptions().then(console.log);
```

### 흐름과 결과

1. `readSetting`이 Promise를 반환합니다.
2. `await`가 성공값을 `setting`에 저장합니다.
3. 가공한 객체를 반환하면 async 함수의 성공값이 됩니다.
4. 호출자는 `then` 또는 다른 `await`로 객체를 받습니다.

### 실무 활용

API 응답 가공, 로컬 저장소 조회, 권한 확인처럼 Promise 결과를 받은 뒤 일반 로직을 이어 가는 함수에 적합합니다.

## 직접 해보기

`getGreeting(name)`을 async 함수로 작성해 `"안녕하세요, 이름"`을 성공값으로 반환하세요. 호출부에서는 `then`으로 출력합니다.

<details>
<summary>답</summary>

```javascript
async function getGreeting(name) {
  return `안녕하세요, ${name}`;
}

getGreeting("민아").then(console.log);
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| 일반 함수 vs async 함수 | 일반 함수는 값을 그대로 반환하지만 async 함수는 Promise로 반환합니다. |
| `return value` vs `await promise` | return은 함수의 결과를 정하고, await는 함수 내부에서 Promise 결과를 꺼내 다음 줄로 이어 갑니다. |
| 함수 전체 중단 vs async 함수 일부 중단 | await는 자바스크립트 전체가 아니라 현재 async 함수의 나머지만 미룹니다. |

## 연결되는 개념

- 여러 await의 배치는 [순차 실행과 병렬 실행](02-sequential-and-parallel-workflows.md)에서 이어집니다.
- await 실패는 [비동기 오류 처리와 전파](03-async-error-handling.md)에서 다룹니다.
- 용어가 낯설다면 [용어집](GLOSSARY.md)을 확인하세요.

## 셀프 체크

- [ ] async 함수의 실제 반환형을 설명할 수 있다.
- [ ] await가 멈추는 범위를 알고 있다.
- [ ] async 함수의 예외가 rejected Promise가 되는 것을 안다.

## 복습 질문 및 답변

### Q1. async 함수에서 숫자 5를 반환하면 호출자는 무엇을 받는가?

<details>
<summary>답</summary>

숫자 5로 이행된 Promise를 받습니다. 실제 숫자는 `await` 또는 `then`으로 얻습니다.

</details>

### Q2. await가 실행되는 동안 다른 자바스크립트 코드는 모두 멈추는가?

<details>
<summary>답</summary>

아닙니다. 현재 async 함수의 나머지 실행만 미뤄지고, 호출 스택이 비면 다른 태스크와 마이크로태스크가 실행될 수 있습니다.

</details>

### Q3. async 함수 안에서 `throw new Error()`를 실행하면 어떻게 되는가?

<details>
<summary>답</summary>

함수가 반환한 Promise가 rejected 상태가 되며, 호출자는 `catch`나 `try/catch`로 처리할 수 있습니다.

</details>

> 최종 한 줄: async는 결과를 Promise로 통일하고, await는 그 Promise의 결과를 함수 안에서 자연스럽게 이어 줍니다.
