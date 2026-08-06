# Promise 상태와 후속 처리

콜백은 “완료 후 무엇을 할지”를 전달하지만, 여러 단계가 겹치면 성공과 실패의 흐름이 흩어지기 쉽습니다. Promise는 미래의 결과를 하나의 객체로 표현해 후속 처리를 연결합니다.

## 핵심 키워드

Promise · pending · fulfilled · rejected · resolve · reject · finally

## 핵심 요약

- Promise는 생성 직후 `pending`이며 성공하면 `fulfilled`, 실패하면 `rejected`가 됩니다.
- `resolve(value)`의 값은 `then`, `reject(reason)`의 이유는 `catch`로 전달됩니다.
- Promise는 한 번 정착되면 다시 상태가 바뀌지 않습니다.
- `finally`는 성공과 실패에 관계없이 정리 작업을 실행합니다.

## 1. Promise 만들기

```javascript
function checkStock(quantity) {
  return new Promise((resolve, reject) => {
    if (quantity > 0) {
      resolve({ available: quantity });
    } else {
      reject(new Error("재고가 없습니다."));
    }
  });
}
```

`new Promise`에 전달한 실행자 함수는 Promise 생성 시 동기적으로 실행됩니다. 다만 `then`이나 `catch`에 등록한 후속 콜백은 현재 동기 코드가 끝난 뒤 실행됩니다.

> 한 줄 정리: Promise는 비동기 작업 자체라기보다, 아직 정해지지 않은 성공값 또는 실패 이유를 담을 계약입니다.

## 2. 성공, 실패, 정리 처리

```javascript
checkStock(3)
  .then((result) => {
    console.log(`남은 수량: ${result.available}`);
  })
  .catch((error) => {
    console.error(error.message);
  })
  .finally(() => {
    console.log("재고 확인 종료");
  });
```

- `then`: 성공값을 처리합니다.
- `catch`: 앞선 체인에서 발생한 거부나 예외를 처리합니다.
- `finally`: 로딩 표시 해제처럼 결과와 무관한 정리를 수행합니다.

`finally` 콜백은 일반적으로 앞의 성공값이나 실패 이유를 바꾸지 않고 다음 단계로 통과시킵니다. 단, `finally` 내부에서 오류를 던지면 체인은 실패로 바뀝니다.

## 3. 한 번만 결정되는 상태

```javascript
const result = new Promise((resolve, reject) => {
  resolve("첫 결과");
  reject(new Error("늦은 실패"));
  resolve("두 번째 결과");
});

result.then(console.log);
```

첫 `resolve`만 유효하므로 `첫 결과`가 전달됩니다. 이러한 불변성 덕분에 소비자는 완료 결과가 뒤집힐 가능성을 걱정하지 않아도 됩니다.

### 자주 헷갈리는 점

- `resolve`는 “성공 로그 출력”이 아니라 Promise의 상태와 결과를 결정하는 함수입니다.
- `return new Promise(...)`를 빠뜨리면 호출자는 `then`을 연결할 수 없습니다.
- 실패는 문자열보다 `Error` 객체로 전달하면 스택과 메시지를 보존하기 좋습니다.

## 대표 코드: 타이머를 Promise로 감싸기

### 목적

지연 완료를 콜백 대신 Promise로 표현해 다른 비동기 단계와 연결합니다.

```javascript
function delay(ms, value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(value);
    }, ms);
  });
}

delay(200, "준비 완료")
  .then((message) => console.log(message))
  .finally(() => console.log("타이머 정리"));
```

### 흐름

1. `delay`는 pending Promise를 즉시 반환합니다.
2. 실행 환경이 타이머를 처리합니다.
3. 콜백에서 `resolve(value)`가 호출되어 fulfilled가 됩니다.
4. 등록된 `then`이 값과 함께 실행됩니다.

### 결과와 실무 활용

재시도 간격, 애니메이션 단계, 테스트 대기 도우미처럼 “시간이 지나면 완료”되는 작업을 일관된 Promise 인터페이스로 다룰 수 있습니다.

## 연습 문제

숫자가 짝수면 해당 숫자를 성공값으로, 홀수면 `Error`를 실패 이유로 전달하는 `validateEven(number)`를 작성해 보세요.

<details>
<summary>답</summary>

```javascript
function validateEven(number) {
  return new Promise((resolve, reject) => {
    if (number % 2 === 0) {
      resolve(number);
      return;
    }

    reject(new Error("짝수가 아닙니다."));
  });
}
```

</details>

## 비교: 콜백 vs Promise

| 구분 | 콜백 | Promise |
|---|---|---|
| 결과 표현 | 함수 호출 | 상태를 가진 객체 |
| 성공·실패 | 별도 규칙 필요 | `then`·`catch`로 분리 |
| 연속 처리 | 중첩되기 쉬움 | 체인으로 연결 |
| 조합 기능 | 직접 구현 | `all` 등 표준 메서드 |

## 연결되는 개념

- [Promise 체이닝과 async/await](05-promise-chaining-and-async-await.md)
- [마이크로태스크와 Promise 조합](06-microtasks-and-promise-combinators.md)
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] Promise의 세 상태를 구분한다.
- [ ] `resolve`와 `reject`가 전달하는 값을 안다.
- [ ] `finally`에 적합한 정리 작업을 설명할 수 있다.

## 복습 질문 및 답변

### Q1. Promise는 `resolve` 후 `reject`로 바뀔 수 있는가?

<details>
<summary>답</summary>

아닙니다. 처음 fulfilled 또는 rejected로 정착된 뒤에는 상태와 결과가 바뀌지 않습니다.

</details>

### Q2. `new Promise`의 실행자 함수는 언제 실행되는가?

<details>
<summary>답</summary>

Promise 객체를 생성하는 시점에 동기적으로 실행됩니다. `then`에 등록한 후속 콜백은 별도의 마이크로태스크로 실행됩니다.

</details>

### Q3. `finally`는 언제 사용하는가?

<details>
<summary>답</summary>

성공과 실패에 상관없이 로딩 표시 해제, 임시 자원 정리처럼 반드시 수행해야 하는 후처리에 사용합니다.

</details>

> 최종 한 줄: Promise는 미래 결과를 pending에서 fulfilled 또는 rejected로 한 번만 결정되는 객체로 표현합니다.
