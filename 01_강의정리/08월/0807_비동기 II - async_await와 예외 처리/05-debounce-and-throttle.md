# 디바운싱과 쓰로틀링

입력, 스크롤, 창 크기 변경 이벤트는 짧은 시간에 수십 번 발생할 수 있습니다. 모든 이벤트마다 무거운 작업을 실행하기보다 호출 빈도를 제어하면 성능과 서버 부하를 함께 개선할 수 있습니다.

## 핵심 키워드

디바운싱 · 쓰로틀링 · 타이머 ID · 마지막 호출 · 호출 간격

## 핵심 요약

- 디바운싱은 연속 이벤트가 멈춘 뒤 마지막 호출을 실행합니다.
- 쓰로틀링은 일정 시간 구간에 최대 한 번 실행합니다.
- 디바운싱은 이전 타이머를 취소해야 합니다.
- 두 패턴 모두 인자와 `this`, 취소 정책을 고려해야 합니다.

## 1. 디바운싱

### 정의와 필요성

사용자가 검색어를 계속 입력하는 동안에는 요청을 미루고, 입력이 잠시 멈췄을 때 마지막 검색어만 요청하는 패턴입니다.

```javascript
function debounce(callback, wait) {
  let timerId;

  return function (...args) {
    clearTimeout(timerId);

    timerId = setTimeout(() => {
      callback.apply(this, args);
    }, wait);
  };
}
```

### 실행 흐름

1. 래퍼 함수가 호출될 때 이전 타이머를 취소합니다.
2. 새 타이머를 등록합니다.
3. 대기 시간 동안 추가 호출이 없으면 콜백을 실행합니다.
4. 결과적으로 연속 호출 중 마지막 값이 반영됩니다.

> 한 줄 정리: 디바운싱은 “조용해진 뒤 마지막 한 번”을 실행합니다.

## 2. 쓰로틀링

### 정의와 필요성

스크롤 위치 표시처럼 이벤트가 계속되는 동안에도 주기적으로 상태를 갱신해야 하지만, 매 이벤트를 처리할 필요는 없을 때 사용합니다.

```javascript
function throttle(callback, interval) {
  let waiting = false;

  return function (...args) {
    if (waiting) {
      return;
    }

    waiting = true;
    callback.apply(this, args);

    setTimeout(() => {
      waiting = false;
    }, interval);
  };
}
```

첫 호출은 즉시 실행하고, 제한 구간 동안의 추가 호출은 무시합니다. 구현 정책에 따라 구간 마지막 호출도 실행하도록 확장할 수 있습니다.

### 자주 헷갈리는 점

- 디바운싱에서 `clearTimeout`을 빠뜨리면 모든 예약이 실행됩니다.
- 쓰로틀링은 마지막 이벤트 값이 반드시 반영된다는 보장이 없습니다.
- 래퍼 안에서 화살표 함수만 무분별하게 사용하면 호출 시점의 `this` 전달이 달라질 수 있습니다.

## 대표 코드: 검색어 입력 제어

### 목적

연속 입력이 끝난 뒤 한 번만 검색 함수를 호출합니다.

```javascript
function requestSuggestions(keyword) {
  console.log(`추천 검색: ${keyword}`);
}

const searchLater = debounce(requestSuggestions, 300);

searchLater("as");
searchLater("asy");
searchLater("async");
```

### 결과와 실무 활용

추가 입력이 없으면 마지막 값인 `async`만 처리됩니다. 자동완성, 폼 검증, 리사이즈 후 레이아웃 계산에 적합합니다. 스크롤 진행률처럼 이벤트 도중에도 갱신이 필요하면 쓰로틀링이 더 알맞습니다.

## 직접 해보기

첫 호출을 즉시 실행하고 500ms 동안 추가 호출을 무시하는 `throttle` 함수를 작성하세요.

<details>
<summary>답</summary>

```javascript
function throttle(callback, interval) {
  let blocked = false;

  return function (...args) {
    if (blocked) {
      return;
    }

    blocked = true;
    callback.apply(this, args);

    setTimeout(() => {
      blocked = false;
    }, interval);
  };
}
```

</details>

## 헷갈리기 쉬운 차이점

| 비교 | 차이 |
|---|---|
| 디바운싱 vs 쓰로틀링 | 멈춘 뒤 마지막 한 번 vs 일정 간격당 최대 한 번 |
| 타이머 재등록 vs 실행 잠금 | 디바운싱은 이전 예약을 취소하고, 기본 쓰로틀은 제한 시간 동안 실행을 막습니다. |
| 검색 입력 vs 스크롤 | 입력 완료 후 요청에는 디바운싱, 진행 중 주기적 갱신에는 쓰로틀링이 흔히 적합합니다. |

## 연결되는 개념

- 타이머의 기본 원리는 이전 학습의 이벤트 루프 개념과 연결됩니다.
- 콜백의 오류를 Promise 흐름으로 다루려면 [비동기 오류 처리와 전파](03-async-error-handling.md)를 참고하세요.
- [용어집](GLOSSARY.md)

## 셀프 체크

- [ ] 디바운싱과 쓰로틀링의 실행 시점을 구분한다.
- [ ] 디바운싱에서 이전 타이머를 취소할 수 있다.
- [ ] 이벤트 성격에 맞는 패턴을 선택할 수 있다.

## 복습 질문 및 답변

### Q1. 검색 자동완성에 디바운싱이 자주 쓰이는 이유는?

<details>
<summary>답</summary>

사용자가 입력을 이어 가는 동안 불필요한 요청을 취소하고, 잠시 멈췄을 때 마지막 검색어만 처리할 수 있기 때문입니다.

</details>

### Q2. 스크롤 위치를 계속 표시해야 할 때 디바운싱만 사용하면 어떤 문제가 있는가?

<details>
<summary>답</summary>

스크롤이 멈출 때까지 갱신되지 않아 진행 중인 위치를 보여 주지 못할 수 있습니다. 일정 간격으로 실행하는 쓰로틀링이 더 적합할 수 있습니다.

</details>

### Q3. 디바운싱에서 clearTimeout을 호출하지 않으면 어떻게 되는가?

<details>
<summary>답</summary>

이전 예약이 취소되지 않아 호출할 때마다 등록한 콜백이 모두 실행되므로 디바운싱 효과가 사라집니다.

</details>

> 최종 한 줄: 디바운싱은 마지막 호출을 기다리고, 쓰로틀링은 실행 빈도에 상한을 둡니다.
