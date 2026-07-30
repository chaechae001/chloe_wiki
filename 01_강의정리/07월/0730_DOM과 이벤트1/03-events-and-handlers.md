# 이벤트와 핸들러 등록

> 이벤트는 사용자의 행동이고, 이벤트 핸들러는 그 행동을 서비스의 반응으로 번역하는 함수입니다.

`event` · `handler` · `addEventListener` · `callback` · `function reference`

## 핵심요약

- 클릭, 입력, 키 누름, 마우스 이동, 문서 로드는 모두 이벤트입니다.
- 이벤트 핸들러는 특정 이벤트가 발생했을 때 브라우저가 호출하는 함수입니다.
- HTML 속성 방식은 구조와 동작이 섞이므로 유지보수에 불리합니다.
- `addEventListener()`는 여러 핸들러 등록과 제거가 가능해 일반적으로 가장 유연합니다.
- 등록할 때 함수 실행 결과가 아니라 함수 참조를 전달해야 합니다.

## 1. 이벤트와 핸들러

### 1) 정의

이벤트는 브라우저에서 관찰되는 사건이고, 이벤트 핸들러는 해당 사건에 반응하도록 등록한 함수입니다. 브라우저는 이벤트가 발생할 때 등록된 함수를 호출합니다.

### 2) 왜 필요한가

버튼 클릭 전에는 창을 열지 않고, 입력이 바뀔 때만 검증하며, 폼이 제출될 때만 서버 요청을 보내야 합니다. 이벤트는 “언제 실행할지”를 결정합니다.

### 3) 핵심 흐름 재구성

1. 반응할 DOM 요소를 선택합니다.
2. 수행할 로직을 함수로 만듭니다.
3. 이벤트 이름과 함수 참조를 등록합니다.
4. 이벤트가 발생하면 브라우저가 함수를 호출합니다.

### 4) 쉬운 예시

초인종을 누르는 것이 `click` 이벤트이고, 문을 열어 주는 행동이 핸들러입니다. 초인종 설치와 행동 규칙을 연결하는 과정이 이벤트 리스너 등록입니다.

### 5) 코드 예시

```javascript
const button = document.querySelector("#toggle-help");
const help = document.querySelector("#help");

function toggleHelp() {
  help.classList.toggle("is-open");
}

button.addEventListener("click", toggleHelp);
```

### 6) 헷갈리는 점

`toggleHelp()`를 전달하면 등록 시점에 함수가 즉시 실행되고 반환값이 전달됩니다. 클릭할 때 실행하려면 괄호 없는 `toggleHelp`를 전달합니다.

### 7) 한 줄 정리

> 이벤트 처리는 대상을 선택하고, 반응 함수를 만든 뒤, 사건 이름과 함수 참조를 연결하는 과정입니다.

## 2. 등록 방식 비교

### 1) 정의

이벤트 핸들러는 HTML 속성, DOM 프로퍼티, `addEventListener()`로 등록할 수 있습니다.

| 방식 | 예시 | 특징 |
|---|---|---|
| HTML 속성 | `<button onclick="openMenu()">` | 구조와 동작이 섞임 |
| DOM 프로퍼티 | `button.onclick = openMenu` | 같은 프로퍼티에 하나만 유지 |
| 이벤트 리스너 | `button.addEventListener("click", openMenu)` | 여러 핸들러와 제거 지원 |

### 2) 왜 필요한가

규모가 커지면 분석 기록, 접근성 처리, 화면 변경처럼 한 이벤트에 여러 관심사가 연결될 수 있습니다. `addEventListener()`를 사용하면 기능을 함수 단위로 분리할 수 있습니다.

### 3) 핵심 흐름 재구성

`on` 프로퍼티 이름에는 `onclick`처럼 접두사가 붙지만 `addEventListener()`의 첫 인수에는 `"click"`처럼 `on`을 제외한 이벤트 이름을 전달합니다.

### 4) 쉬운 예시

DOM 프로퍼티는 게시판에 공지 한 장만 붙이는 것과 비슷합니다. 새 공지를 붙이면 이전 공지가 가려집니다. 이벤트 리스너는 신청자 명단에 여러 담당자를 추가하는 방식입니다.

### 5) 코드 예시

```javascript
function showFeedback() {
  document.querySelector("#feedback").textContent = "선택되었습니다.";
}

function recordInteraction() {
  console.log("interaction: plan-selected");
}

const planButton = document.querySelector("#plan-basic");
planButton.addEventListener("click", showFeedback);
planButton.addEventListener("click", recordInteraction);
```

### 6) 헷갈리는 점

`window.onload = handler`도 프로퍼티 방식이므로 나중에 다른 함수를 대입하면 교체됩니다. 문서 준비 시점을 다룰 때는 `document.addEventListener("DOMContentLoaded", handler)`도 사용할 수 있습니다.

### 7) 한 줄 정리

> 일반적인 애플리케이션 코드에서는 동작을 분리하기 쉬운 `addEventListener()`가 기본 선택입니다.

## 코드로 보기 — 여러 카드에 같은 핸들러 등록

```javascript
const cards = document.querySelectorAll(".choice-card");

function handleChoice() {
  this.classList.toggle("is-selected");
}

cards.forEach((card) => {
  card.addEventListener("click", handleChoice);
});
```

### 코드 목적

여러 카드 각각을 클릭할 때 해당 카드의 선택 상태만 토글합니다.

### 코드 흐름

1. 카드 전체를 선택합니다.
2. 재사용할 핸들러를 한 번 정의합니다.
3. 각 카드에 같은 함수 참조를 등록합니다.
4. 호출 시 일반 함수의 `this`는 리스너가 등록된 요소를 가리킵니다.

### 실행 결과 해석

첫 번째 카드를 클릭하면 첫 번째 카드에만 `is-selected` 클래스가 추가됩니다. 다시 클릭하면 제거됩니다.

### 실무 연결

옵션 카드, 좋아요 버튼, 아코디언 항목처럼 같은 동작을 공유하는 여러 요소에 재사용할 수 있습니다. 더 많은 요소에는 이벤트 위임도 고려합니다.

## 직접 해보기

1. `#start` 버튼 클릭 시 `startTimer` 함수가 실행되도록 등록해 보세요.
2. `startTimer()`를 리스너의 두 번째 인수로 전달하면 생기는 문제를 설명해 보세요.
3. 같은 클릭에 화면 변경과 로그 기록을 모두 연결할 방법을 적어 보세요.

<details><summary>정답 보기</summary>

1. `document.querySelector("#start").addEventListener("click", startTimer);`입니다.
2. 등록 순간 함수가 실행되고 그 반환값이 핸들러 자리에 전달됩니다.
3. 두 함수를 각각 `addEventListener("click", 함수참조)`로 등록하거나 하나의 조정 함수에서 차례로 호출합니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 이벤트 vs 핸들러 | 이벤트는 발생한 사건, 핸들러는 그 사건에 반응하는 함수입니다. |
| `handler` vs `handler()` | 앞은 함수 참조, 뒤는 함수를 즉시 호출한 표현입니다. |
| `onclick` vs `addEventListener()` | 프로퍼티는 한 값을 유지하고 리스너 방식은 여러 함수를 등록할 수 있습니다. |
| `"click"` vs `"onclick"` | 리스너의 이벤트 이름에는 `on`을 붙이지 않습니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [요소 조작과 동적 생성](02-dom-manipulation-and-creation.md)
- 다음에 이어지는 개념: [event 객체와 상호작용 패턴](04-event-object-and-interactions.md)
- 함께 보면 좋은 키워드: `콜백`, `상태`, `DOMContentLoaded`

## 셀프 체크

- [ ] 이벤트와 이벤트 핸들러를 구분할 수 있다.
- [ ] 함수 참조를 올바르게 등록할 수 있다.
- [ ] 세 가지 등록 방식의 차이를 설명할 수 있다.
- [ ] 여러 요소에 같은 핸들러를 등록할 수 있다.
- [ ] 클래스 토글로 상호작용 상태를 표현할 수 있다.

### 복습 질문 및 답변

**Q1. `addEventListener()`의 두 인수는 무엇인가요?**

<details><summary>답</summary>

첫 번째는 `"click"` 같은 이벤트 이름, 두 번째는 실행할 핸들러 함수 참조입니다.

</details>

**Q2. DOM 프로퍼티 방식에서 함수를 두 번 대입하면 어떻게 되나요?**

<details><summary>답</summary>

같은 프로퍼티의 이전 함수가 새 함수로 교체됩니다.

</details>

**Q3. 반복되는 카드 UI에서 이벤트 코드를 줄이는 첫 단계는 무엇인가요?**

<details><summary>답</summary>

공통 동작을 하나의 이름 있는 핸들러로 만들고 반복문으로 각 카드에 같은 참조를 등록합니다.

</details>

## 한 줄 정리

> `addEventListener()`로 DOM 요소의 사건과 재사용 가능한 함수 참조를 연결하면 인터페이스가 반응하기 시작합니다.

