# 이벤트 전파의 세 단계

> 가장 안쪽 버튼을 한 번 클릭했는데 부모의 핸들러도 실행된다면, 이벤트가 DOM 경로를 이동하고 있기 때문입니다.

`propagation` · `capturing` · `target phase` · `bubbling` · `eventPhase`

## 핵심요약

- 중첩된 DOM에서 하나의 이벤트는 대상과 조상 요소로 이루어진 경로를 이동합니다.
- 캡처링은 바깥에서 안으로, 버블링은 안에서 바깥으로 진행됩니다.
- 실제 발생 대상에서는 타깃 단계가 됩니다.
- 일반적인 리스너는 버블링 단계에서 실행되며 캡처 옵션으로 캡처링을 선택할 수 있습니다.
- 전파 중단은 필요한 경우에만 제한적으로 사용해야 다른 기능을 막지 않습니다.

## 1. 하나의 이벤트가 여러 요소에서 보이는 이유

### 1) 정의

이벤트 전파는 클릭 같은 하나의 이벤트가 대상 요소와 그 조상 요소들로 구성된 경로를 따라 전달되는 과정입니다. 각 요소에서 별도 클릭이 새로 만들어지는 것이 아닙니다.

### 2) 왜 필요한가

브라우저가 계층 구조를 따라 이벤트를 전달하기 때문에 부모는 자식에서 일어난 행동을 감지할 수 있습니다. 메뉴, 표, 목록처럼 비슷한 자식이 많은 UI를 효율적으로 처리하는 기반이 됩니다.

### 3) 핵심 흐름 재구성

```html
<main id="app">
  <section id="panel">
    <button id="save">저장</button>
  </section>
</main>
```

`#save`를 클릭하면 이벤트 경로에는 대략 `window → document → html → body → #app → #panel → #save`가 포함됩니다. 내려가는 캡처링, 대상에 도착하는 타깃, 다시 올라가는 버블링이 차례로 진행됩니다.

### 4) 쉬운 예시

아파트의 안쪽 집에 택배가 도착한다고 생각해 봅시다. 입구에서 집까지 내려가는 과정이 캡처링, 수령자에게 전달되는 순간이 타깃, 전달 사실이 관리실 방향으로 보고되는 과정이 버블링과 비슷합니다.

### 5) 코드 예시

```javascript
const panel = document.querySelector("#panel");
const saveButton = document.querySelector("#save");

panel.addEventListener("click", () => console.log("panel"));
saveButton.addEventListener("click", () => console.log("button"));
```

버튼을 클릭하면 기본 버블링 리스너에서는 `button`, `panel` 순으로 출력됩니다.

### 6) 헷갈리는 점

부모 핸들러가 실행되는 것은 자식 핸들러가 부모 함수를 직접 호출해서가 아닙니다. 두 핸들러가 같은 이벤트 경로의 서로 다른 위치에서 하나의 이벤트를 관찰한 결과입니다.

### 7) 한 줄 정리

> 이벤트 전파는 하나의 사건이 DOM 조상 경로를 따라 이동하는 브라우저의 전달 규칙입니다.

## 2. 캡처링·타깃·버블링

### 1) 정의

이벤트 전파는 세 단계로 구분됩니다.

| 단계 | `event.eventPhase` | 방향과 의미 |
|---|---:|---|
| 캡처링 | 1 | 상위 객체에서 대상의 부모 방향으로 내려감 |
| 타깃 | 2 | 이벤트가 실제 대상에 도착함 |
| 버블링 | 3 | 대상의 부모에서 상위 객체 방향으로 올라감 |

### 2) 왜 필요한가

실행 순서를 예측하고, 어느 계층에서 이벤트를 관찰할지 결정하려면 현재 단계와 방향을 알아야 합니다. 대부분은 기본 버블링으로 충분하지만 초기 차단이나 특수한 관찰에는 캡처링이 쓰일 수 있습니다.

### 3) 핵심 흐름 재구성

`addEventListener(type, listener, true)` 또는 `{ capture: true }`는 캡처링에서 실행됩니다. 세 번째 인수를 생략하거나 `false`로 두면 기본 버블링에서 실행됩니다.

### 4) 쉬운 예시

공연장 입장 시 입구에서 좌석까지 확인하는 흐름이 캡처링이라면, 공연 후 좌석에서 출구로 나가는 흐름은 버블링입니다. 실제 좌석에서 관람하는 순간이 타깃입니다.

### 5) 코드 예시

```javascript
const app = document.querySelector("#app");
const button = document.querySelector("#save");

app.addEventListener("click", logPhase, { capture: true });
button.addEventListener("click", logPhase);
app.addEventListener("click", logPhase);

function logPhase(event) {
  console.log(event.currentTarget.id, event.eventPhase);
}
```

### 6) 헷갈리는 점

캡처링과 버블링은 이벤트 종류에 따라 동작 차이가 있습니다. 모든 이벤트가 같은 방식으로 버블링한다고 가정하지 말고 사용하는 이벤트의 특성을 확인해야 합니다.

### 7) 한 줄 정리

> 단계는 이벤트가 현재 경로의 어느 방향과 위치에 있는지를 알려 줍니다.

## 코드로 보기 — 전파 순서 직접 확인하기

```javascript
const outer = document.querySelector("#outer");
const inner = document.querySelector("#inner");

function report(label) {
  return function (event) {
    console.log(label, {
      phase: event.eventPhase,
      target: event.target.id,
      current: event.currentTarget.id
    });
  };
}

outer.addEventListener("click", report("outer capture"), true);
inner.addEventListener("click", report("inner target"));
outer.addEventListener("click", report("outer bubble"));
```

### 코드 목적

중첩 요소 클릭 시 단계, 실제 대상, 현재 핸들러 위치를 한 번에 관찰합니다.

### 코드 흐름

1. 바깥 요소에 캡처링 리스너를 등록합니다.
2. 안쪽 요소와 바깥 요소에 기본 리스너를 등록합니다.
3. 안쪽 요소를 클릭합니다.
4. 콘솔의 단계와 실행 순서를 비교합니다.

### 실행 결과 해석

바깥 캡처 리스너가 먼저, 안쪽 타깃 리스너가 다음, 바깥 버블 리스너가 마지막에 실행됩니다. 세 출력의 `target`은 같지만 `current`는 실행 위치에 따라 달라집니다.

### 실무 연결

모달 바깥 클릭 감지, 분석 이벤트 수집, 동적 목록 제어처럼 부모 계층에서 자식 행동을 관찰할 때 전파 구조를 활용합니다.

## 직접 해보기

1. 캡처링과 버블링의 진행 방향을 각각 적어 보세요.
2. 캡처링 리스너를 등록하는 세 번째 인수를 작성해 보세요.
3. 부모와 자식에 기본 클릭 리스너가 있을 때 자식 클릭의 출력 순서를 예측해 보세요.

<details><summary>정답 보기</summary>

1. 캡처링은 상위에서 대상 방향, 버블링은 대상에서 상위 방향입니다.
2. `true` 또는 `{ capture: true }`를 사용합니다.
3. 자식의 타깃 리스너가 먼저 실행되고 이후 부모의 버블링 리스너가 실행됩니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 이벤트 발생 vs 이벤트 전파 | 발생은 사건의 시작이고 전파는 그 사건이 DOM 경로를 이동하는 과정입니다. |
| 캡처링 vs 버블링 | 상위→대상 방향인지 대상→상위 방향인지 다릅니다. |
| `preventDefault()` vs `stopPropagation()` | 기본 브라우저 행동 취소와 이벤트 이동 중단은 서로 다른 제어입니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: DOM 트리와 이벤트 핸들러
- 다음에 이어지는 개념: [이벤트 대상과 위임](02-event-targets-and-delegation.md)
- 함께 보면 좋은 키워드: `이벤트 경로`, `리스너`, `DOM 계층`

## 셀프 체크

- [ ] 하나의 이벤트가 여러 요소에서 감지되는 이유를 설명할 수 있다.
- [ ] 세 전파 단계의 순서를 말할 수 있다.
- [ ] 캡처링 리스너를 등록할 수 있다.
- [ ] `eventPhase` 값을 해석할 수 있다.
- [ ] 전파 중단과 기본 동작 취소를 구분할 수 있다.

### 복습 질문 및 답변

**Q1. 기본 `addEventListener()`는 어느 단계에서 동작하나요?**

<details><summary>답</summary>

세 번째 인수를 생략하면 기본적으로 버블링 단계에서 이벤트를 감지합니다.

</details>

**Q2. 이벤트가 부모마다 새로 생성되나요?**

<details><summary>답</summary>

아닙니다. 하나의 이벤트가 정해진 DOM 경로를 이동하며 각 리스너가 이를 관찰합니다.

</details>

**Q3. 전파를 무조건 막으면 안 되는 이유는 무엇인가요?**

<details><summary>답</summary>

상위 요소의 단축키, 닫기 처리, 분석 수집 등 같은 이벤트에 의존하는 다른 기능까지 실행되지 않을 수 있기 때문입니다.

</details>

## 한 줄 정리

> 캡처링·타깃·버블링의 경로를 이해하면 중첩된 UI의 이벤트 실행 순서를 예측할 수 있습니다.

