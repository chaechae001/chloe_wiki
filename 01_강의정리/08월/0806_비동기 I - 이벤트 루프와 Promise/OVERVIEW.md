# 비동기 I: 이벤트 루프와 Promise

자바스크립트는 한 번에 하나의 작업을 실행하지만, 타이머와 네트워크 요청까지 멈춰서 기다리지는 않습니다. 이 학습노트는 **호출 스택–이벤트 루프–큐**의 협업 구조를 이해하고, 콜백과 Promise로 비동기 결과를 안전하게 이어 가는 방법을 익히는 데 초점을 둡니다.

## 학습 목표

- 동기 처리와 비동기 처리의 차이를 실행 흐름으로 설명한다.
- `setTimeout`, `setInterval`의 예약과 취소 방식을 구분한다.
- 호출 스택, Web API, 태스크 큐, 마이크로태스크 큐의 역할을 연결한다.
- Promise의 세 상태와 `then`, `catch`, `finally`의 동작을 설명한다.
- Promise 체이닝, `async`/`await`, `Promise.all`을 상황에 맞게 사용한다.
- 코드만 보고 로그 출력 순서를 예측한다.

## 추천 학습 순서

1. [타이머와 비동기 예약](01-timers-and-scheduling.md)
2. [싱글 스레드와 이벤트 루프](02-single-thread-and-event-loop.md)
3. [태스크 큐와 실행 순서](03-task-queue-and-execution-order.md)
4. [Promise 상태와 후속 처리](04-promise-states-and-handlers.md)
5. [Promise 체이닝과 async/await](05-promise-chaining-and-async-await.md)
6. [마이크로태스크와 Promise 조합](06-microtasks-and-promise-combinators.md)
7. [용어집](GLOSSARY.md)

## 전체 흐름 한눈에 보기

```text
동기 코드 → 호출 스택에서 즉시 실행
타이머 등록 → 실행 환경이 시간 측정 → 콜백을 태스크 큐에 대기
Promise 후속 처리 → 마이크로태스크 큐에 대기
호출 스택이 비면 → 마이크로태스크를 먼저 비움 → 태스크 하나 실행
```

## 학습 체크리스트

- [ ] 지연 시간이 0ms여도 콜백이 즉시 실행되지 않는 이유를 말할 수 있다.
- [ ] `clearTimeout`과 `clearInterval`에 무엇을 전달해야 하는지 안다.
- [ ] `resolve`와 `reject`가 Promise 상태를 어떻게 바꾸는지 안다.
- [ ] `then`에서 반환한 값이 다음 `then`으로 전달되는 과정을 설명할 수 있다.
- [ ] Promise 콜백이 타이머 콜백보다 먼저 실행되는 일반적인 이유를 안다.
- [ ] 여러 비동기 작업의 순차 실행과 병렬 시작을 구분할 수 있다.

> 핵심 질문: “이 코드는 언제 실행되는가?”와 “완료 결과를 어디로 전달하는가?”를 함께 추적하면 비동기 코드는 훨씬 선명해집니다.
