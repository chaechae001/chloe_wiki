# 브라우저 내장 객체: window와 document

> JavaScript가 실행되는 환경을 이해하면 어떤 객체가 언어 기능이고 어떤 객체가 브라우저 기능인지 구분할 수 있습니다.

`built-in object` · `window` · `document` · `viewport` · `popup`

## 핵심요약

- 내장 객체는 반복적으로 필요한 기능을 미리 구현해 제공하는 객체입니다.
- 브라우저에서 `window`는 전역 객체이자 현재 창을 나타냅니다.
- `document`는 현재 HTML 문서를 나타내며 `window.document`로도 접근할 수 있습니다.
- 창 크기·URL·타이머는 `window`, DOM 탐색·조작은 `document`의 책임입니다.
- 새 창 열기는 사용자 동작과 브라우저 보안 정책의 영향을 받습니다.

## 1. 실행 환경과 내장 객체

### 1) 정의

내장 객체는 언어 또는 실행 환경이 기본 제공하는 프로퍼티와 메서드의 묶음입니다. `Math`, `Date`, `JSON`은 JavaScript 표준에 가깝고, `window`와 `document`는 브라우저 환경이 제공합니다.

### 2) 왜 필요한가

브라우저 창 크기를 읽는 코드와 수학 계산 코드는 출처와 실행 가능 환경이 다릅니다. Node.js 같은 다른 환경에는 브라우저의 `window`가 없으므로 객체의 소속을 알아야 코드를 올바르게 배치할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
console.log(window.innerWidth);
console.log(document.title);
console.log(window.document === document); // true
```

브라우저 전역 범위에서 일부 `window` 프로퍼티는 `window.`를 생략할 수 있지만, 학습 단계에서는 소속을 명시하면 역할이 더 잘 보입니다.

### 4) 쉬운 예시

브라우저 탭을 하나의 사무실이라고 보면 `window`는 사무실 전체와 창문·출입문 같은 시설이고, `document`는 사무실 안의 문서와 가구 배치도입니다.

### 5) 코드 예시

```javascript
function readEnvironment() {
  return {
    width: window.innerWidth,
    height: window.innerHeight,
    url: document.URL,
    title: document.title
  };
}

console.table(readEnvironment());
```

### 6) 헷갈리는 점

`window`는 JavaScript 언어 어디서나 항상 존재하는 객체가 아닙니다. 브라우저가 아닌 실행 환경에서 참조하면 `ReferenceError`가 날 수 있습니다.

### 7) 한 줄 정리

> 내장 객체를 사용할 때는 그 기능이 언어에서 왔는지 실행 환경에서 왔는지 먼저 구분합니다.

## 2. 창 기능과 문서 기능의 분리

### 1) 정의

`window`는 화면 크기, 현재 위치, 새 창, 타이머 같은 브라우저 수준 기능을 제공합니다. `document`는 요소 선택, 생성, 속성 변경처럼 HTML 문서 작업을 담당합니다.

### 2) 왜 필요한가

책임을 구분하면 API를 찾기 쉽습니다. 현재 뷰포트 너비는 `window.innerWidth`, 페이지 제목은 `document.title`, 버튼 선택은 `document.querySelector()`를 사용합니다.

### 3) 핵심 흐름 재구성

| 목적 | 시작 객체 | 예시 |
|---|---|---|
| 콘텐츠 영역 크기 | `window` | `window.innerWidth` |
| 새 탐색 컨텍스트 | `window` | `window.open()` |
| 문서 주소·제목 | `document` | `document.URL`, `document.title` |
| DOM 요소 선택 | `document` | `document.querySelector()` |

### 4) 쉬운 예시

건물의 크기나 출입구를 확인하는 일과 방 안의 책상을 옮기는 일은 담당 부서가 다릅니다. 창 기능과 문서 기능도 같은 방식으로 나뉩니다.

### 5) 코드 예시

```javascript
const openHelpButton = document.querySelector("#open-help");

openHelpButton.addEventListener("click", () => {
  const features = "width=520,height=640,resizable=yes";
  const helpWindow = window.open("/help", "help", features);

  if (!helpWindow) {
    console.warn("팝업이 차단되었습니다.");
  }
});
```

### 6) 헷갈리는 점

`window.open()`은 보통 사용자 클릭과 직접 연결되어야 팝업 차단 가능성이 낮습니다. `window.close()`도 스크립트가 연 창이 아닌 경우 브라우저가 닫기를 허용하지 않을 수 있습니다.

### 7) 한 줄 정리

> 창과 브라우저 상태는 `window`, HTML 구조는 `document`에서 다룹니다.

## 코드로 보기 — 화면 크기에 따라 안내 갱신하기

```javascript
const viewportLabel = document.querySelector("#viewport-label");

function updateViewportLabel() {
  const compact = window.innerWidth < 768;
  viewportLabel.textContent = compact ? "작은 화면" : "넓은 화면";
  document.body.classList.toggle("is-compact", compact);
}

window.addEventListener("resize", updateViewportLabel);
updateViewportLabel();
```

### 코드 목적

브라우저 콘텐츠 영역 너비를 읽어 안내 문구와 문서의 상태 클래스를 동기화합니다.

### 코드 흐름

1. `window.innerWidth`로 현재 너비를 읽습니다.
2. 768px 미만인지 불리언으로 계산합니다.
3. `document`에서 선택한 요소의 텍스트와 클래스를 바꿉니다.
4. 크기 변경 이벤트와 초기 실행에 같은 함수를 사용합니다.

### 실행 결과 해석

창이 좁아지면 “작은 화면”과 `is-compact` 클래스가 적용되고, 넓어지면 반대 상태가 됩니다.

### 실무 연결

반응형 내비게이션, 차트 크기 재계산, 편집기 레이아웃 조정에서 뷰포트 정보를 사용할 수 있습니다. CSS만으로 해결 가능한 표현은 미디어 쿼리를 우선합니다.

## 직접 해보기

1. 현재 문서 제목을 출력하는 코드를 작성해 보세요.
2. `window.innerWidth`와 `document.documentElement.clientWidth`가 모두 너비와 관련 있지만 목적이 다를 수 있는 이유를 생각해 보세요.
3. 새 창 열기 코드가 자동 실행보다 버튼 클릭 안에서 잘 작동하는 이유를 설명해 보세요.

<details><summary>정답 보기</summary>

1. `console.log(document.title);`입니다.
2. 브라우저 UI와 스크롤바 포함 여부 등 측정 기준이 다를 수 있으므로 필요한 영역 기준을 선택해야 합니다.
3. 브라우저가 원치 않는 팝업을 막기 때문에 사용자의 명시적 동작과 직접 연결된 호출을 더 신뢰하기 때문입니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| JavaScript 표준 객체 vs 브라우저 객체 | 언어 자체에서 제공되는지 브라우저 환경에서 제공되는지 다릅니다. |
| `window` vs `document` | 창·전역 환경과 HTML 문서라는 책임이 다릅니다. |
| 새 탭 vs 새 창 | `window.open()`의 표시 방식은 옵션뿐 아니라 브라우저 정책과 사용자 설정에도 좌우됩니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: DOM 선택과 이벤트
- 다음에 이어지는 개념: [Number와 Math](02-number-and-math.md)
- 함께 보면 좋은 키워드: `BOM`, `DOM`, `실행 환경`

## 셀프 체크

- [ ] 내장 객체의 의미를 설명할 수 있다.
- [ ] 브라우저 객체와 표준 객체를 구분할 수 있다.
- [ ] `window`와 `document`의 책임을 구분할 수 있다.
- [ ] 뷰포트 크기를 읽을 수 있다.
- [ ] 팝업 관련 브라우저 제한을 설명할 수 있다.

### 복습 질문 및 답변

**Q1. `document`는 어디에 연결되어 있나요?**

<details>
<summary>답</summary>

브라우저 전역 객체인 `window`의 `document` 프로퍼티이며 현재 HTML 문서를 나타냅니다.

</details>

**Q2. Node.js에서 `window`가 없을 수 있는 이유는 무엇인가요?**

<details>
<summary>답</summary>

`window`는 JavaScript 언어 자체가 아니라 브라우저 창 환경이 제공하는 객체이기 때문입니다.

</details>

**Q3. 초기 화면 상태에도 반응형 함수를 호출해야 하는 이유는 무엇인가요?**

<details>
<summary>답</summary>

`resize`는 크기가 바뀔 때만 발생하므로 페이지가 처음 열린 크기에 맞는 상태를 별도로 설정해야 합니다.

</details>

## 한 줄 정리

> 실행 환경의 객체 책임을 구분하면 브라우저 상태와 HTML 문서를 올바른 API로 다룰 수 있습니다.
