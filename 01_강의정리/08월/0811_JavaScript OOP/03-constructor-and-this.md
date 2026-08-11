# 생성자와 this

생성자는 인스턴스가 태어날 때 필요한 값을 초기화하고, `this`는 지금 메서드를 실행하고 있는 인스턴스를 가리킵니다.

## 핵심 키워드

`constructor` · `this` · `매개변수` · `인스턴스 프로퍼티` · `new`

## 핵심 요약

- `constructor`는 `new`로 인스턴스를 만들 때 한 번 실행됩니다.
- 생성자 매개변수는 입력값이고 `this.property`는 인스턴스에 저장되는 상태입니다.
- 인스턴스 메서드에서 `this`는 호출한 객체를 기준으로 결정됩니다.
- 파생 클래스 생성자에서는 `this`보다 먼저 `super()`를 호출해야 합니다.

## 1. 생성자의 초기화 책임

생성자는 객체가 유효한 초기 상태로 시작하도록 값을 설정합니다.

```javascript
class Account {
  constructor(owner, balance = 0) {
    this.owner = owner;
    this.balance = balance;
  }
}

const account = new Account("사용자", 10000);
console.log(account.balance); // 10000
```

`balance`는 함수가 받은 매개변수이고 `this.balance`는 새 인스턴스에 저장되는 프로퍼티입니다. 이름이 같아도 역할이 다릅니다.

생성자를 생략한 기본 클래스에는 빈 생성자가 있는 것처럼 동작합니다. 초기화 규칙이 필요할 때 명시적으로 작성합니다.

## 2. this의 의미

일반 메서드의 `this`는 메서드가 어떻게 호출됐는지에 따라 정해집니다.

```javascript
class Account {
  constructor(owner, balance = 0) {
    this.owner = owner;
    this.balance = balance;
  }

  describe() {
    const balance = "지역 변수";
    return `${this.owner}: ${this.balance}원`;
  }
}
```

`balance`는 함수 내부 지역 변수이고 `this.balance`는 객체 상태입니다. 프로퍼티임을 명확하게 표현하려면 `this`가 필요합니다.

## 3. 메서드 분리와 this 손실

메서드를 객체에서 떼어 별도 함수처럼 호출하면 원래 객체와의 호출 관계가 사라질 수 있습니다.

```javascript
const account = new Account("사용자", 10000);
const describe = account.describe;

// describe(); // 엄격 모드에서는 this가 undefined여서 오류가 날 수 있습니다.
const boundDescribe = account.describe.bind(account);
console.log(boundDescribe());
```

이벤트 콜백이나 고차 함수에 메서드를 전달할 때는 `bind`, 래퍼 함수, 화살표 함수 필드 중 상황에 맞는 방법을 선택합니다.

## 대표 코드: 유효한 초기 상태 만들기

### 목적

생성 단계에서 입력을 검증해 잘못된 객체가 생기지 않게 합니다.

```javascript
class Subscription {
  constructor(plan, months) {
    if (!plan.trim()) {
      throw new Error("요금제가 필요합니다.");
    }

    if (!Number.isInteger(months) || months < 1) {
      throw new Error("이용 개월은 1 이상의 정수여야 합니다.");
    }

    this.plan = plan;
    this.months = months;
  }

  extend(extraMonths) {
    this.months += extraMonths;
  }
}

const subscription = new Subscription("basic", 3);
subscription.extend(1);
console.log(subscription.months); // 4
```

### 코드 흐름과 결과

1. 생성자가 입력값을 검사합니다.
2. 검증을 통과한 값만 인스턴스에 저장합니다.
3. 메서드는 `this`로 현재 객체의 상태를 변경합니다.
4. 객체는 최소 1개월이라는 규칙을 지닌 상태로 시작합니다.

### 실무 연결

생성자는 주문, 예약, 설정 객체처럼 필수 값과 초기 규칙이 있는 도메인 객체의 진입점이 됩니다.

## 직접 해보기

1. 생성자 매개변수와 인스턴스 프로퍼티의 차이를 설명하세요.
2. `extend`에서 1 미만의 값을 거절하도록 수정하세요.
3. 객체 메서드를 콜백으로 바로 전달할 때 생길 수 있는 문제를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 매개변수는 생성자가 받은 입력이고 프로퍼티는 객체에 지속적으로 저장되는 상태입니다.
2. `if (!Number.isInteger(extraMonths) || extraMonths < 1) throw new Error("잘못된 기간");`을 먼저 검사합니다.
3. 호출 주체가 사라져 `this`가 원래 인스턴스를 가리키지 않을 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 매개변수 `name` vs `this.name` | 앞은 함수 입력, 뒤는 현재 인스턴스의 프로퍼티입니다. |
| 지역 변수 vs 인스턴스 프로퍼티 | 지역 변수는 호출 동안만 존재하고 프로퍼티는 객체 상태로 남습니다. |
| 일반 함수의 `this` vs 화살표 함수의 `this` | 일반 함수는 호출 방식, 화살표 함수는 선언된 주변 렉시컬 환경을 따릅니다. |

## 연결되는 개념

- 인스턴스의 기본 구조는 [클래스와 인스턴스](02-classes-and-instances.md)에서 설명합니다.
- 변경 규칙을 통제하는 방법은 [접근자와 상태 검증](04-accessors-and-validation.md)에서 이어집니다.
- 파생 클래스의 초기화는 [상속과 super, 오버라이딩](06-inheritance-super-and-overriding.md)에서 다룹니다.

## 셀프 체크

- [ ] 생성자가 실행되는 시점을 설명할 수 있다.
- [ ] 매개변수와 `this` 프로퍼티를 구분할 수 있다.
- [ ] 메서드를 분리할 때의 `this` 문제를 알고 있다.

## 복습 질문 및 답변

### Q1. `constructor`는 인스턴스 생성 때 몇 번 실행되는가?

<details>
<summary>답</summary>

각 `new` 호출마다 한 번 실행됩니다.

</details>

### Q2. 생성자를 작성하지 않으면 클래스로 객체를 만들 수 없는가?

<details>
<summary>답</summary>

만들 수 있습니다. 기본 클래스에는 인수를 사용하지 않는 기본 생성자가 있는 것처럼 동작합니다.

</details>

### Q3. 메서드 안에서 `this.value` 대신 `value`만 쓰면 같은 프로퍼티를 읽는가?

<details>
<summary>답</summary>

아닙니다. `value`는 현재 스코프의 식별자를 찾고, 객체 프로퍼티는 `this.value`로 명시해야 합니다.

</details>

## 한 줄 정리

> 생성자는 객체의 시작 상태를 만들고 `this`는 그 상태를 가진 현재 인스턴스를 메서드와 연결합니다.
