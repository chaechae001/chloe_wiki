# `this`와 호출 방식

> `this`는 함수를 소유한 곳에 영원히 붙어 있는 값이 아니라, 일반 함수가 어떻게 호출됐는지를 보여주는 실행 시점의 단서입니다.

`this` · `method call` · `default binding` · `object state` · `method chaining`

## 핵심요약

- 일반 함수의 `this`는 주로 함수가 호출되는 방식에 따라 결정됩니다.
- `object.method()` 형태에서는 `this`가 점 앞의 객체를 가리킵니다.
- 메서드를 변수에 분리해 일반 함수로 호출하면 원래 객체와의 연결을 잃을 수 있습니다.
- 브라우저 전역·일반 함수의 `this`는 엄격 모드와 실행 환경에 따라 다릅니다.
- 메서드가 `this`를 반환하면 동일 객체의 다음 메서드를 이어서 호출할 수 있습니다.

## 1. 메서드 호출의 `this`

### 1) 정의

객체의 프로퍼티로 저장된 함수를 객체를 통해 호출하면 그 호출에서 `this`는 메서드를 호출한 객체를 가리킵니다.

### 2) 왜 필요한가

객체 이름을 코드에 직접 적지 않고도 현재 객체의 프로퍼티를 읽거나 바꿀 수 있어 같은 메서드 로직을 여러 객체에서 재사용할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
const profile = {
  nickname: "miso",
  introduce() {
    return `안녕하세요, ${this.nickname}입니다.`;
  },
};

console.log(profile.introduce());
```

호출식의 점 앞에 있는 `profile`이 `this`의 기준이 됩니다.

### 4) 쉬운 예시

같은 자기소개 대본을 여러 사람이 읽더라도 “내 이름”은 대본이 아니라 실제로 대본을 읽는 사람에 따라 달라지는 것과 같습니다.

### 5) 코드 예시

```javascript
function getLabel() {
  return `${this.category}: ${this.name}`;
}

const book = { category: "도서", name: "JavaScript", getLabel };
const course = { category: "강의", name: "Web", getLabel };

console.log(book.getLabel());   // 도서: JavaScript
console.log(course.getLabel()); // 강의: Web
```

### 6) 헷갈리는 점

함수가 객체 안에 작성됐다는 사실보다 실제 호출식이 중요합니다. 같은 함수도 다른 객체의 메서드로 호출하면 `this`가 달라집니다.

### 7) 한 줄 정리

> 메서드 호출에서 `this`는 현재 그 메서드를 호출한 객체입니다.

## 2. 분리 호출과 실행 환경

### 1) 정의

메서드를 객체에서 꺼내 별도 변수로 호출하면 `object.method()` 구조가 사라져 일반 함수 호출이 됩니다.

### 2) 왜 필요한가

콜백으로 메서드를 전달하거나 함수를 구조 분해할 때 발생하는 `this` 손실을 예측하려면 호출식을 확인해야 합니다.

### 3) 핵심 흐름 재구성

```javascript
const cart = {
  count: 3,
  getCount() {
    return this.count;
  },
};

const detached = cart.getCount;
// detached(); // 엄격 모드에서는 this가 undefined여서 오류 가능
```

### 4) 쉬운 예시

사원증을 회사 밖에 따로 두면 어느 직원의 것인지 연결 정보가 사라지는 상황과 비슷합니다.

### 5) 코드 예시

```javascript
"use strict";

function inspectThis() {
  return this;
}

console.log(inspectThis()); // undefined
```

브라우저의 비엄격 스크립트, ES 모듈, Node.js에서는 전역 `this`와 일반 함수 `this`가 서로 다르게 보일 수 있으므로 `window`라고 단정하지 않는 편이 안전합니다.

### 6) 헷갈리는 점

`this`와 렉시컬 스코프는 다릅니다. 일반 함수의 변수 검색은 정의 위치를 따르지만 `this`는 호출 방식의 영향을 받습니다.

### 7) 한 줄 정리

> 메서드를 분리하면 원래 객체를 통한 호출 정보도 함께 사라질 수 있습니다.

## 코드로 보기 — 객체 상태와 메서드 체이닝

```javascript
const player = {
  score: 0,

  add(points) {
    this.score += points;
    return this;
  },

  reset() {
    this.score = 0;
    return this;
  },

  current() {
    return this.score;
  },
};

const result = player.add(10).add(5).current();
console.log(result); // 15
```

### 코드 목적

메서드에서 현재 객체의 상태를 변경하고 `this`를 반환해 연속 호출을 구성합니다.

### 코드 흐름

1. `player.add(10)`에서 `this`는 `player`가 됩니다.
2. 점수를 바꾸고 같은 객체를 반환합니다.
3. 반환된 객체에서 다시 `add(5)`를 호출합니다.
4. 마지막 조회 메서드는 숫자 결과를 반환하며 체인을 끝냅니다.

### 실행 결과 해석

두 번의 상태 변경이 같은 객체에 누적되어 최종 점수는 15입니다.

### 실무 연결

설정 빌더, 쿼리 작성기, 화면 컴포넌트의 상태 조작처럼 여러 작업을 순서대로 연결하는 API 설계에 활용할 수 있습니다.

## 직접 해보기

1. `user.print()`에서 `this`는 무엇을 가리키나요?
2. `const print = user.print; print()`처럼 호출하면 무엇이 달라지나요?
3. 변경 메서드가 `return this` 대신 숫자를 반환하면 메서드 체이닝은 어떻게 되나요?

<details>
<summary>정답 보기</summary>

1. 일반 함수 메서드라면 호출식의 점 앞 객체인 `user`를 가리킵니다.
2. 객체를 통한 호출이 사라져 `this`가 `user`로 바인딩되지 않습니다.
3. 반환된 숫자에는 다음 객체 메서드가 없으므로 같은 형태의 체이닝을 이어갈 수 없습니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 객체에 저장된 함수 vs 메서드 호출 | 저장 위치만으로는 부족하며 실제로 객체를 통해 호출해야 해당 객체가 `this`가 됩니다. |
| `this` vs 객체 이름 직접 참조 | `this`는 호출 객체에 따라 재사용할 수 있지만 객체 이름 직접 참조는 특정 객체에 결합됩니다. |
| 상태 변경 반환값 vs `return this` | 값을 반환하면 결과 사용에, `this`를 반환하면 연속 메서드 호출에 적합합니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [콜 스택과 스코프 체인](02-call-stack-scope-chain-and-closures.md)
- 다음에 이어지는 개념: [`call`·`apply`·`bind`](04-call-apply-and-bind.md)
- 함께 보면 좋은 키워드: `객체 리터럴`, `엄격 모드`, `메서드 체이닝`

## 셀프 체크

- [ ] 메서드 호출에서 `this`를 찾을 수 있다.
- [ ] 같은 함수를 여러 객체에서 재사용할 수 있다.
- [ ] 메서드 분리 호출의 문제를 설명할 수 있다.
- [ ] 실행 환경에 따라 기본 바인딩이 달라질 수 있음을 안다.
- [ ] `return this`로 체이닝을 구성할 수 있다.

### 복습 질문 및 답변

**Q1. `this`는 항상 함수가 들어 있는 객체를 가리키나요?**

<details>
<summary>답</summary>

아닙니다. 일반 함수에서는 실제 호출 방식이 중요하며, 객체를 통하지 않고 호출하면 그 객체와의 바인딩이 없습니다.

</details>

**Q2. 객체 이름 대신 `this`를 쓰면 어떤 장점이 있나요?**

<details>
<summary>답</summary>

함수가 특정 객체 이름에 결합되지 않아 다른 객체의 메서드로도 재사용할 수 있습니다.

</details>

**Q3. 모든 변경 메서드가 `this`를 반환해야 하나요?**

<details>
<summary>답</summary>

아닙니다. 연속 호출이 API의 목적일 때 유용하며, 변경된 값이나 성공 여부가 더 중요하면 그 결과를 반환하는 편이 낫습니다.

</details>

## 한 줄 정리

> 일반 함수의 `this`를 이해하려면 함수의 작성 위치보다 실제 호출식과 반환값 설계를 먼저 확인해야 합니다.
