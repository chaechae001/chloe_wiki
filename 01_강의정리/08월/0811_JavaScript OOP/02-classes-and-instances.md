# 클래스와 인스턴스

클래스는 객체를 반복해서 만들기 위한 정의이고, 인스턴스는 그 정의에 실제 값을 담아 생성한 독립적인 객체입니다.

## 핵심 키워드

`class` · `instance` · `new` · `property` · `method` · `instanceof`

## 핵심 요약

- 클래스는 객체의 공통 구조와 행동을 정의합니다.
- `new`는 클래스를 바탕으로 새 인스턴스를 만듭니다.
- 각 인스턴스는 자신의 프로퍼티 값을 독립적으로 가집니다.
- 메서드는 객체가 수행할 수 있는 행동을 표현합니다.

## 1. 클래스와 인스턴스의 관계

클래스는 재사용 가능한 설계 정의입니다. 인스턴스는 메모리에 생성되어 실제로 값을 갖고 동작하는 객체입니다.

```javascript
class Timer {
  seconds = 0;

  tick() {
    this.seconds += 1;
  }
}

const focusTimer = new Timer();
const breakTimer = new Timer();

focusTimer.tick();
console.log(focusTimer.seconds); // 1
console.log(breakTimer.seconds); // 0
```

두 인스턴스는 같은 구조와 메서드를 사용하지만 상태는 서로 섞이지 않습니다.

## 2. 프로퍼티와 메서드

프로퍼티는 객체의 현재 상태를 나타내고, 메서드는 상태를 읽거나 변경하는 행동을 나타냅니다.

| 구성 요소 | 질문 | 예시 |
|---|---|---|
| 프로퍼티 | 객체가 무엇을 알고 있는가? | 남은 시간, 이름, 활성 여부 |
| 메서드 | 객체가 무엇을 할 수 있는가? | 시작, 정지, 값 변경 |

클래스 필드에 화살표 함수를 저장하면 각 인스턴스가 자기 함수 값을 가질 수 있습니다. 일반 메서드와 `this` 동작 및 메모리 특성이 다르므로 필요할 때 선택해야 합니다.

```javascript
class ButtonModel {
  label = "저장";

  handleClick = () => {
    return `${this.label} 실행`;
  };
}
```

## 3. 인스턴스 확인

`instanceof`는 특정 생성자의 프로토타입이 객체의 프로토타입 체인에 있는지 확인합니다.

```javascript
console.log(focusTimer instanceof Timer); // true
console.log({} instanceof Timer); // false
```

단순히 모양이 같은 객체인지 확인하는 문법은 아닙니다. 서로 다른 실행 환경에서 만들어진 객체나 상속 관계에서는 프로토타입 체인을 기준으로 해석해야 합니다.

## 대표 코드: 재사용 가능한 작업 모델

### 목적

같은 구조를 가진 여러 작업을 독립적인 인스턴스로 관리합니다.

```javascript
class Task {
  title;
  completed = false;

  rename(nextTitle) {
    this.title = nextTitle;
  }

  complete() {
    this.completed = true;
  }
}

const study = new Task();
study.rename("클래스 복습");
study.complete();

const exercise = new Task();
exercise.rename("산책하기");

console.log(study.completed); // true
console.log(exercise.completed); // false
```

### 코드 흐름과 결과

1. `Task`가 공통 프로퍼티와 행동을 정의합니다.
2. `new Task()`를 두 번 호출해 독립 객체를 만듭니다.
3. 같은 메서드가 각자의 상태를 변경합니다.
4. 한 인스턴스의 완료 상태는 다른 인스턴스에 영향을 주지 않습니다.

### 실무 연결

같은 규칙을 가진 데이터를 여러 개 생성해야 할 때 클래스가 생성 규칙과 동작을 한곳에 모아 줍니다.

## 직접 해보기

1. 클래스와 인스턴스의 차이를 설명하세요.
2. `Task`에 완료 상태를 되돌리는 `reopen()`을 추가하세요.
3. 모든 작업을 하나의 전역 객체로 관리할 때의 문제를 생각해 보세요.

<details>
<summary>정답 보기</summary>

1. 클래스는 공통 정의이고 인스턴스는 그 정의로 생성한 실제 객체입니다.
2. `reopen() { this.completed = false; }`로 작성할 수 있습니다.
3. 서로 다른 작업의 상태가 섞이고 변경 책임을 추적하기 어려워집니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 클래스 vs 인스턴스 | 클래스는 정의, 인스턴스는 `new`로 만든 실제 객체입니다. |
| 프로퍼티 vs 메서드 | 프로퍼티는 상태값, 메서드는 행동을 나타냅니다. |
| 일반 메서드 vs 화살표 함수 필드 | 일반 메서드는 프로토타입에 공유되고 화살표 함수 필드는 인스턴스마다 생성됩니다. |

## 연결되는 개념

- 클래스 설계 관점은 [객체 지향 프로그래밍의 핵심 원리](01-oop-principles.md)에서 확인할 수 있습니다.
- 생성 시 값을 초기화하는 법은 [생성자와 this](03-constructor-and-this.md)에서 이어집니다.
- 내부 상태 접근은 [접근자와 상태 검증](04-accessors-and-validation.md)에서 다룹니다.

## 셀프 체크

- [ ] 클래스와 인스턴스를 구분할 수 있다.
- [ ] 프로퍼티와 메서드를 설계할 수 있다.
- [ ] 인스턴스별 상태가 독립적임을 설명할 수 있다.

## 복습 질문 및 답변

### Q1. `new Task()`를 두 번 호출하면 같은 객체를 가리키는가?

<details>
<summary>답</summary>

아닙니다. 같은 클래스를 바탕으로 하지만 서로 다른 독립 인스턴스를 생성합니다.

</details>

### Q2. 클래스 필드 `completed = false`는 언제 각 객체에 생기는가?

<details>
<summary>답</summary>

새 인스턴스가 생성될 때 각 인스턴스에 초기값으로 만들어집니다.

</details>

### Q3. 클래스 없이도 객체 지향 설계를 할 수 있는가?

<details>
<summary>답</summary>

가능합니다. 객체 리터럴, 생성자 함수, 프로토타입 조합 등으로도 상태와 행동의 책임을 객체에 배치할 수 있습니다.

</details>

## 한 줄 정리

> 클래스는 공통 규칙을 재사용하고 인스턴스는 그 규칙 아래 서로 독립적인 상태를 갖습니다.
