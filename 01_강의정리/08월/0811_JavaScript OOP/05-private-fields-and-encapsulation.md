# private 필드와 캡슐화

캡슐화는 단순히 값을 숨기는 일이 아니라, 객체가 유효한 상태를 지키도록 허용된 행동만 외부에 공개하는 설계입니다.

## 핵심 키워드

`encapsulation` · `#private` · `public interface` · `invariant` · `private method`

## 핵심 요약

- `#` private 요소는 선언한 클래스 외부에서 직접 접근할 수 없습니다.
- private 필드는 상속받은 자식 클래스에서도 이름으로 직접 접근할 수 없습니다.
- 공개 메서드와 접근자는 내부 상태를 사용하는 안전한 통로가 됩니다.
- 캡슐화의 목표는 객체가 지켜야 할 규칙을 보호하는 것입니다.

## 1. 언어 수준의 비공개 상태

밑줄 접두사는 개발자 사이의 관례지만 `#` private 필드는 자바스크립트 문법이 접근을 제한합니다.

```javascript
class Wallet {
  #balance = 0;

  deposit(amount) {
    if (amount <= 0) {
      throw new RangeError("입금액은 0보다 커야 합니다.");
    }
    this.#balance += amount;
  }

  get balance() {
    return this.#balance;
  }
}

const wallet = new Wallet();
wallet.deposit(5000);
console.log(wallet.balance); // 5000
// console.log(wallet.#balance); // 클래스 바깥에서는 문법 오류
```

private 이름은 클래스 본문에 선언되어 있어야 하며 점 표기법이나 대괄호 표기법으로 우회할 수 없습니다.

## 2. 불변 조건 보호

객체가 항상 지켜야 하는 규칙을 불변 조건이라고 생각할 수 있습니다. 지갑 잔액이 검증된 행동으로만 바뀌게 만들면 음수 입금 같은 잘못된 상태를 차단할 수 있습니다.

```javascript
class Wallet {
  #balance = 0;

  deposit(amount) {
    this.#assertPositive(amount);
    this.#balance += amount;
  }

  withdraw(amount) {
    this.#assertPositive(amount);
    if (amount > this.#balance) {
      throw new RangeError("잔액이 부족합니다.");
    }
    this.#balance -= amount;
  }

  #assertPositive(amount) {
    if (!Number.isFinite(amount) || amount <= 0) {
      throw new RangeError("금액은 0보다 큰 숫자여야 합니다.");
    }
  }

  get balance() {
    return this.#balance;
  }
}
```

검증 로직도 private 메서드로 숨겨 공개 사용법을 단순하게 만들 수 있습니다.

## 3. 상속과 private

private 필드는 선언한 클래스의 내부에서만 접근할 수 있습니다. 자식 클래스는 부모가 공개한 getter, setter, 메서드를 통해 기능을 사용해야 합니다.

이 제한은 상속을 방해하는 결함이 아니라 부모 클래스의 내부 구현을 보호하는 경계입니다. 자식이 반드시 접근해야 하는 상태라면 공개 또는 보호용 메서드 설계를 다시 검토해야 합니다.

## 대표 코드: 상태 변경 통로 제한

### 목적

서비스 활성 상태를 직접 수정하지 못하게 하고 공개 행동으로만 전환합니다.

```javascript
class ServiceSession {
  #active = false;
  #startedAt = null;

  start(now = new Date()) {
    if (this.#active) return false;
    this.#active = true;
    this.#startedAt = now;
    return true;
  }

  stop() {
    if (!this.#active) return false;
    this.#active = false;
    return true;
  }

  isActive() {
    return this.#active;
  }

  get startedAt() {
    return this.#startedAt;
  }
}

const session = new ServiceSession();
console.log(session.start()); // true
console.log(session.isActive()); // true
```

### 코드 흐름과 결과

1. private 필드가 상태를 외부 직접 대입에서 보호합니다.
2. `start`와 `stop`이 허용된 상태 전환만 수행합니다.
3. `isActive`는 불리언 질문에 답하는 공개 메서드입니다.
4. 중복 시작과 중복 종료는 `false`로 결과를 알립니다.

### 실무 연결

결제 상태, 재고 수량, 인증 세션처럼 잘못된 순서의 변경을 막아야 하는 객체에서 행동 중심 인터페이스가 중요합니다.

## 직접 해보기

1. 밑줄 필드와 private 필드의 차이를 설명하세요.
2. `ServiceSession`에 시작 시각을 초기화하는 `reset()`을 설계하세요.
3. private 필드를 모두 getter와 setter로 공개하면 캡슐화가 약해질 수 있는 이유를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 밑줄은 접근 자제 관례지만 private 필드는 문법적으로 외부 접근이 제한됩니다.
2. 허용할 정책에 따라 비활성 상태에서만 `#startedAt = null`로 바꾸게 만들 수 있습니다.
3. 외부가 내부 상태를 자유롭게 읽고 쓰면 객체가 행동과 순서를 통제하지 못해 규칙이 다시 외부로 흩어집니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 정보 은닉 vs 캡슐화 | 은닉은 노출 제한, 캡슐화는 상태와 규칙을 묶어 안전한 인터페이스를 만드는 더 넓은 개념입니다. |
| `_field` vs `#field` | 밑줄은 관례, `#`은 언어가 강제하는 private 이름입니다. |
| private vs 상속 가능 | private 요소는 자식 클래스에서도 직접 접근할 수 없고 부모의 공개 기능을 사용해야 합니다. |

## 연결되는 개념

- 값 검증 인터페이스는 [접근자와 상태 검증](04-accessors-and-validation.md)에서 확인할 수 있습니다.
- 부모의 공개 기능을 재사용하는 법은 [상속과 super, 오버라이딩](06-inheritance-super-and-overriding.md)에서 이어집니다.
- 전체 OOP 관점은 [객체 지향 프로그래밍의 핵심 원리](01-oop-principles.md)에서 설명합니다.

## 셀프 체크

- [ ] private 필드의 접근 범위를 설명할 수 있다.
- [ ] 객체가 지켜야 할 상태 규칙을 정의할 수 있다.
- [ ] 공개 인터페이스를 행동 중심으로 설계할 수 있다.

## 복습 질문 및 답변

### Q1. 자식 클래스가 부모의 `#balance`를 직접 읽을 수 있는가?

<details>
<summary>답</summary>

읽을 수 없습니다. 부모 클래스가 제공한 공개 또는 허용된 메서드와 접근자를 사용해야 합니다.

</details>

### Q2. private 필드를 사용하면 자동으로 좋은 캡슐화가 완성되는가?

<details>
<summary>답</summary>

아닙니다. 어떤 행동을 공개하고 어떤 상태 전환을 허용할지 책임 있게 설계해야 합니다.

</details>

### Q3. 불리언 상태를 묻는 메서드에 `is` 접두사를 쓰는 장점은?

<details>
<summary>답</summary>

반환값이 참·거짓인 질문이라는 의도를 호출부에서 빠르게 이해할 수 있습니다.

</details>

## 한 줄 정리

> private 요소와 행동 중심 인터페이스는 객체의 상태 규칙을 내부에서 지키게 하는 캡슐화 도구입니다.
