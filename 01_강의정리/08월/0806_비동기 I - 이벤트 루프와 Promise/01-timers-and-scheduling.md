# 타이머와 비동기 예약

“2초 뒤 실행”은 자바스크립트가 2초 동안 멈춘다는 뜻이 아닙니다. 실행 환경에 콜백을 맡기고 다음 코드를 계속 실행한다는 뜻입니다.

## 핵심 키워드

`setTimeout` · `setInterval` · 타이머 ID · 최소 지연 시간 · 콜백

## 핵심 요약

- `setTimeout`은 콜백을 일정 시간 뒤 **한 번** 실행하도록 예약합니다.
- `setInterval`은 콜백을 일정 간격으로 **반복** 예약합니다.
- 두 함수의 반환값인 타이머 ID를 보관하면 예약을 취소할 수 있습니다.
- 지연 시간은 정확한 실행 시각이 아니라 실행 가능한 최소 대기 시간입니다.

## 1. 타이머는 왜 필요한가

알림 자동 닫기, 검색어 입력 지연 처리, 주기적 상태 확인처럼 “지금이 아닌 나중”에 수행할 일이 있습니다. 이때 시간을 직접 재는 반복문을 사용하면 호출 스택을 점유해 화면과 입력이 멈춥니다. 타이머 API는 시간 측정을 실행 환경에 맡겨 현재 흐름을 막지 않습니다.

```javascript
console.log("저장 요청");

setTimeout(() => {
  console.log("잠시 후 상태 확인");
}, 300);

console.log("다음 작업");
```

실행 흐름은 `저장 요청 → 다음 작업 → 잠시 후 상태 확인`입니다. 타이머 콜백은 등록만 되고, 현재 동기 코드가 끝난 후 실행 기회를 얻습니다.

> 한 줄 정리: 타이머는 기다리는 코드가 아니라, 나중에 실행할 함수를 예약하는 API입니다.

## 2. 한 번 실행과 반복 실행

```javascript
const noticeTimer = setTimeout(() => {
  console.log("알림을 닫습니다.");
}, 500);

const heartbeatTimer = setInterval(() => {
  console.log("연결 상태 확인");
}, 1000);

clearTimeout(noticeTimer);
clearInterval(heartbeatTimer);
```

`clearTimeout`과 `clearInterval`은 콜백 함수가 아니라 예약할 때 받은 ID를 사용합니다. 반복 타이머는 종료 조건을 빠뜨리면 계속 실행되므로 생성 지점과 정리 지점을 함께 설계해야 합니다.

### 자주 헷갈리는 점

- `setTimeout(handler(), 1000)`은 함수를 즉시 호출한 결과를 전달합니다. 함수 참조 `handler` 또는 `() => handler()`를 전달해야 합니다.
- `1000`은 1초이며 단위는 밀리초입니다.
- `setInterval`의 콜백 수행 시간이 간격보다 길면 실행 시점이 밀릴 수 있습니다.

## 대표 코드: 취소 가능한 지연 작업

### 목적

사용자가 연속으로 검색어를 바꿀 때 이전 요청 예약을 취소하고 마지막 입력만 처리합니다.

```javascript
let pendingTimer;

function scheduleSearch(keyword) {
  clearTimeout(pendingTimer);

  pendingTimer = setTimeout(() => {
    console.log(`검색 시작: ${keyword}`);
  }, 250);
}

scheduleSearch("pro");
scheduleSearch("promise");
```

### 흐름

1. 이전 타이머 ID로 기존 예약을 취소합니다.
2. 새 검색어를 처리할 타이머를 등록합니다.
3. 추가 입력이 없을 때 마지막 콜백만 실행됩니다.

### 결과와 실무 활용

예제에서는 `검색 시작: promise`만 출력됩니다. 검색 자동완성, 입력 검증, 창 크기 변경 처리처럼 빈번한 이벤트를 줄이는 데 활용할 수 있습니다.

## 연습 문제

`scheduleMessage(message, seconds)`가 `seconds`초 후 메시지를 출력하고, 예약을 취소할 수 있도록 타이머 ID를 반환하게 작성해 보세요.

<details>
<summary>답</summary>

```javascript
function scheduleMessage(message, seconds) {
  return setTimeout(() => {
    console.log(message);
  }, seconds * 1000);
}

const timerId = scheduleMessage("완료", 2);
clearTimeout(timerId);
```

</details>

## 비교: setTimeout vs setInterval

| 구분 | setTimeout | setInterval |
|---|---|---|
| 실행 횟수 | 한 번 | 취소할 때까지 반복 |
| 취소 함수 | `clearTimeout` | `clearInterval` |
| 적합한 상황 | 지연 실행, 디바운스 | 주기적 갱신, 타이머 표시 |
| 주의점 | 실행 시각은 보장되지 않음 | 종료 조건과 중복 등록 관리 |

## 연결되는 개념

- 타이머 콜백의 다음 목적지는 [태스크 큐와 실행 순서](03-task-queue-and-execution-order.md)에서 다룹니다.
- 타이머 완료를 값으로 표현하는 방법은 [Promise 상태와 후속 처리](04-promise-states-and-handlers.md)에서 이어집니다.
- 용어가 낯설다면 [용어집](GLOSSARY.md)을 확인하세요.

## 셀프 체크

- [ ] 함수를 전달하는 것과 호출 결과를 전달하는 것을 구분한다.
- [ ] 타이머 ID를 사용해 예약을 취소할 수 있다.
- [ ] 지연 시간이 정확한 실행 시각을 뜻하지 않는 이유를 안다.

## 복습 질문 및 답변

### Q1. `setTimeout(callback, 0)`은 콜백을 즉시 실행할까?

<details>
<summary>답</summary>

아닙니다. 콜백은 타이머 처리 후 태스크 큐에서 대기하며, 현재 호출 스택의 동기 코드가 모두 끝난 뒤 실행 기회를 얻습니다.

</details>

### Q2. 반복 타이머를 안전하게 종료하려면 무엇이 필요한가?

<details>
<summary>답</summary>

`setInterval`의 반환값인 타이머 ID와 명확한 종료 조건이 필요합니다. 조건을 만족하면 `clearInterval(timerId)`를 호출합니다.

</details>

### Q3. 긴 반복문으로 지연 시간을 만들면 왜 문제인가?

<details>
<summary>답</summary>

호출 스택을 계속 점유하여 다른 함수, 사용자 입력, 화면 갱신이 실행되지 못하기 때문입니다.

</details>

> 최종 한 줄: 타이머는 시간을 보낸 뒤 실행할 콜백을 등록하고, ID로 그 예약을 관리하는 도구입니다.
