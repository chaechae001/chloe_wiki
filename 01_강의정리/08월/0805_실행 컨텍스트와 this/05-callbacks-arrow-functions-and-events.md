# 콜백과 렉시컬 `this`

> 콜백 안의 `this`가 예상과 달라지는 이유는 함수를 전달한 곳과 실제로 호출한 곳이 다르기 때문입니다.

`callback` · `arrow function` · `lexical this` · `event listener` · `currentTarget`

## 핵심요약

- 콜백은 다른 함수가 정한 방식으로 호출되므로 원래 메서드의 호출 정보를 잃을 수 있습니다.
- 일반 콜백의 `this`는 콜백을 실제로 호출한 방식에 따라 결정됩니다.
- 화살표 함수는 자체 `this`를 만들지 않고 정의된 바깥 스코프의 `this`를 사용합니다.
- 바깥 메서드의 `this`를 보존하려면 화살표 함수 또는 `bind()`를 선택할 수 있습니다.
- DOM 이벤트에서 일반 함수의 `this`와 화살표 함수의 `this`는 다르므로 `event.currentTarget`을 명시하면 안전합니다.

## 1. 콜백에서 `this`를 잃는 이유

### 1) 정의

콜백은 다른 함수에 값으로 전달된 뒤 그 함수 내부에서 호출되는 함수입니다. 메서드가 콜백으로 분리되면 원래 객체를 통한 호출 구조가 유지되지 않을 수 있습니다.

### 2) 왜 필요한가

배열 메서드, 타이머, 이벤트, 비동기 처리에서 콜백을 자주 사용하므로 호출 주체를 잘못 예상하면 `undefined`나 `NaN` 같은 결과가 생깁니다.

### 3) 핵심 흐름 재구성

```javascript
function execute(task) {
  return task();
}

const counter = {
  value: 5,
  read() {
    return execute(function () {
      return this.value;
    });
  },
};
```

안쪽 일반 함수는 `task()`로 호출되므로 `counter`의 메서드 호출이 아닙니다.

### 4) 쉬운 예시

회사 대표로 참석하던 사람이 명찰 없이 외부 행사에 개인 자격으로 불리면 원래 회사 정보가 자동으로 연결되지 않는 상황과 비슷합니다.

### 5) 코드 예시

```javascript
function transform(callback) {
  const value = callback();
  return value * 2;
}

const score = {
  base: 40,
  calculate() {
    return transform(() => this.base + 10);
  },
};

console.log(score.calculate()); // 100
```

### 6) 헷갈리는 점

콜백을 메서드 안에서 작성했다고 해서 일반 함수 콜백의 `this`가 자동으로 바깥 메서드와 같아지는 것은 아닙니다.

### 7) 한 줄 정리

> 콜백의 `this`는 전달된 위치가 아니라 실제 콜백 호출 방식에 영향을 받습니다.

## 2. 화살표 함수와 이벤트

### 1) 정의

화살표 함수는 자신의 `this` 바인딩을 만들지 않고 정의된 위치의 바깥 `this`를 참조합니다. 이를 렉시컬 `this`라고 합니다.

### 2) 왜 필요한가

중첩 콜백에서 바깥 메서드의 객체 상태를 그대로 사용하고 싶을 때 별도의 바인딩 코드 없이 의도를 표현할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const timer = {
  seconds: 0,
  start() {
    setTimeout(() => {
      this.seconds += 1;
    }, 1000);
  },
};
```

화살표 콜백의 바깥은 `start` 메서드이고, 그 메서드의 `this`는 `timer`입니다.

### 4) 쉬운 예시

화살표 함수는 별도의 주소를 새로 정하지 않고 자신을 둘러싼 바깥 함수의 주소를 그대로 사용하는 전달원과 같습니다.

### 5) 코드 예시

```javascript
const button = document.querySelector("button");

button.addEventListener("click", (event) => {
  event.currentTarget.textContent = "완료";
});
```

화살표 함수에서 클릭된 요소를 다룰 때는 `this` 대신 `event.currentTarget`을 명시하면 의도가 선명합니다.

### 6) 헷갈리는 점

객체 메서드 자체를 화살표 함수로 작성하면 호출 객체의 `this`를 새로 받지 못합니다. 메서드에는 일반 함수 축약 문법, 그 안의 콜백에는 화살표 함수가 자주 어울립니다.

### 7) 한 줄 정리

> 화살표 함수는 바깥 `this`를 보존하지만 호출 객체에 따라 `this`가 바뀌어야 하는 메서드에는 적합하지 않습니다.

## 코드로 보기 — 안전한 비동기 상태 변경

```javascript
const uploader = {
  completed: 0,

  finishLater(delay) {
    setTimeout(() => {
      this.completed += 1;
      console.log(`완료: ${this.completed}`);
    }, delay);
  },
};

uploader.finishLater(100);
```

### 코드 목적

타이머 콜백이 실행될 때도 바깥 메서드의 객체 상태를 안전하게 변경합니다.

### 코드 흐름

1. `uploader.finishLater()` 호출에서 메서드의 `this`가 `uploader`가 됩니다.
2. 화살표 콜백은 이 `this`를 렉시컬하게 참조합니다.
3. 타이머가 끝난 뒤에도 같은 객체의 `completed`를 증가시킵니다.
4. 변경된 값을 출력합니다.

### 실행 결과 해석

지정한 시간이 지난 뒤 `완료: 1`이 출력됩니다. 콜백 실행 시점이 늦어져도 참조하는 `this`는 유지됩니다.

### 실무 연결

네트워크 응답, 타이머, 이벤트 처리 후 현재 서비스 객체나 컴포넌트 상태를 갱신하는 코드에 연결됩니다.

## 직접 해보기

1. 일반 함수 콜백이 바깥 메서드의 `this`를 자동으로 유지하지 못하는 이유는 무엇인가요?
2. 화살표 함수의 `this`는 어떻게 결정되나요?
3. 버튼 이벤트에서 리스너가 등록된 요소를 명시적으로 사용하려면 무엇을 참조하면 되나요?

<details>
<summary>정답 보기</summary>

1. 콜백은 전달된 뒤 다른 함수의 일반 호출 방식으로 실행될 수 있어 원래 객체를 통한 호출 구조가 사라지기 때문입니다.
2. 자체 바인딩 없이 함수가 정의된 바깥 렉시컬 스코프의 `this`를 사용합니다.
3. 이벤트 객체의 `currentTarget`을 사용하면 됩니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 일반 함수 vs 화살표 함수 | 일반 함수는 호출 방식으로 `this`를 정하고 화살표 함수는 바깥 `this`를 사용합니다. |
| `target` vs `currentTarget` | `target`은 실제 이벤트 발생 요소, `currentTarget`은 리스너가 등록된 요소입니다. |
| 화살표 콜백 vs 화살표 메서드 | 바깥 메서드의 `this` 보존에는 유용하지만 호출 객체의 `this`가 필요한 메서드 자체에는 부적합할 수 있습니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [`call`·`apply`·`bind`](04-call-apply-and-bind.md)
- 다음에 이어지는 개념: [호이스팅과 TDZ](06-hoisting-and-tdz.md)
- 함께 보면 좋은 키워드: `타이머`, `DOM 이벤트`, `비동기 처리`

## 셀프 체크

- [ ] 콜백의 실제 호출 방식을 확인할 수 있다.
- [ ] 일반 함수와 화살표 함수의 `this` 차이를 설명할 수 있다.
- [ ] 콜백에서 `bind()`와 화살표 함수를 선택할 수 있다.
- [ ] 이벤트의 `currentTarget`을 사용할 수 있다.
- [ ] 객체 메서드를 무조건 화살표 함수로 만들지 않는 이유를 안다.

### 복습 질문 및 답변

**Q1. 화살표 함수에 `call()`을 사용하면 `this`가 바뀌나요?**

<details>
<summary>답</summary>

아닙니다. 화살표 함수는 자체 `this`가 없어 `call()`로 새 `this`를 지정할 수 없고 바깥 값을 계속 사용합니다.

</details>

**Q2. 이벤트 리스너에서 화살표 함수를 쓰면 `this`가 버튼인가요?**

<details>
<summary>답</summary>

아닙니다. 화살표 함수는 바깥 `this`를 사용하므로 버튼은 `event.currentTarget`으로 참조하는 편이 명확합니다.

</details>

**Q3. 콜백에서 화살표 함수와 `bind()` 중 무엇을 선택해야 하나요?**

<details>
<summary>답</summary>

바깥 `this`를 그대로 쓸 짧은 콜백에는 화살표 함수가 간결하고, 기존 일반 함수를 재사용하거나 바인딩된 함수를 따로 전달해야 하면 `bind()`가 적합합니다.

</details>

## 한 줄 정리

> 콜백에서는 누가 함수를 호출하는지 확인하고, 바깥 객체의 `this`가 필요하면 화살표 함수나 `bind()`로 의도를 명시합니다.
