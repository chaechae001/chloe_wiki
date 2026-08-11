# 상속과 super, 오버라이딩

상속은 이미 검증된 공통 기능을 이어받아 특수한 클래스로 확장하는 방법이며, `super`와 오버라이딩은 부모 기능을 재사용하고 조정하는 도구입니다.

## 핵심 키워드

`extends` · `inheritance` · `super()` · `super.method()` · `overriding`

## 핵심 요약

- `extends`는 부모 클래스와 자식 클래스의 상속 관계를 만듭니다.
- 자식 생성자에서는 `this`를 사용하기 전에 `super()`를 호출해야 합니다.
- 같은 이름의 메서드를 자식에서 정의하면 부모 메서드를 오버라이딩합니다.
- `super.method()`로 부모 구현을 재사용한 뒤 결과를 확장할 수 있습니다.

## 1. 공통 기능 상속

상속은 자식이 부모의 공개된 기능을 사용할 수 있게 합니다.

```javascript
class Report {
  constructor(title) {
    this.title = title;
  }

  summarize() {
    return { title: this.title };
  }
}

class SalesReport extends Report {
  total = 0;

  setTotal(value) {
    this.total = value;
  }
}
```

`SalesReport` 인스턴스는 자신의 기능뿐 아니라 `summarize()`도 사용할 수 있습니다. 상속은 두 클래스가 실제로 “is-a” 관계일 때 자연스럽습니다.

## 2. 자식 생성자와 super()

자식 클래스가 생성자를 정의하면 부모 초기화를 먼저 수행해야 합니다.

```javascript
class SalesReport extends Report {
  constructor(title, total) {
    super(title);
    this.total = total;
  }
}
```

파생 클래스에서는 `super()`가 현재 인스턴스를 사용할 수 있는 기반을 마련하므로 그 전에 `this`에 접근하면 오류가 발생합니다.

## 3. 오버라이딩과 부모 메서드 재사용

자식 클래스가 부모와 같은 이름의 메서드를 정의하면 자식 구현이 호출됩니다.

```javascript
class SalesReport extends Report {
  constructor(title, total) {
    super(title);
    this.total = total;
  }

  summarize() {
    const base = super.summarize();
    return { ...base, total: this.total };
  }
}

const report = new SalesReport("주간 매출", 120000);
console.log(report.summarize());
```

부모 결과를 객체 전개 구문으로 복사하고 자식 정보를 추가합니다. 부모 구현을 완전히 대체할 수도 있지만 공통 규칙을 보존해야 한다면 `super`로 재사용하는 편이 안전합니다.

## 대표 코드: 알림 유형별 동작 확장

### 목적

공통 메시지 규칙은 부모에 두고 자식이 채널 정보만 확장합니다.

```javascript
class Message {
  constructor(receiver, text) {
    this.receiver = receiver;
    this.text = text;
  }

  toPayload() {
    return { receiver: this.receiver, text: this.text };
  }
}

class EmailMessage extends Message {
  constructor(receiver, text, subject) {
    super(receiver, text);
    this.subject = subject;
  }

  toPayload() {
    return {
      ...super.toPayload(),
      channel: "email",
      subject: this.subject,
    };
  }
}

const message = new EmailMessage("user@example.test", "완료되었습니다.", "작업 알림");
console.log(message.toPayload().channel); // email
```

### 코드 흐름과 결과

1. 부모 생성자가 공통 수신자와 본문을 초기화합니다.
2. 자식 생성자가 채널 고유 상태를 추가합니다.
3. 자식 메서드가 부모 결과를 호출하고 확장합니다.
4. 같은 `toPayload()` 호출이 객체 유형에 맞는 결과를 만듭니다.

### 실무 연결

공통 규칙이 안정적이고 하위 유형이 명확한 메시지, 결제 수단, UI 컴포넌트 모델에서 사용할 수 있습니다. 상속 단계가 깊어지면 결합이 커지므로 조합도 함께 검토합니다.

## 직접 해보기

1. `extends`와 `super()`의 역할을 각각 설명하세요.
2. `SmsMessage`가 `channel: "sms"`를 추가하도록 작성하세요.
3. 부모 메서드가 바뀌었을 때 자식 오버라이딩에 미치는 위험을 설명하세요.

<details>
<summary>정답 보기</summary>

1. `extends`는 상속 관계를 만들고 `super()`는 부모 생성자를 호출해 부모 상태를 초기화합니다.
2. `class SmsMessage extends Message { toPayload() { return { ...super.toPayload(), channel: "sms" }; } }`처럼 작성할 수 있습니다.
3. 자식이 부모의 반환 구조나 부작용에 의존하면 부모 변경이 자식 동작을 깨뜨릴 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| `super()` vs `super.method()` | 앞은 부모 생성자 호출, 뒤는 부모 프로토타입 메서드 호출입니다. |
| 오버라이딩 vs 새 메서드 | 오버라이딩은 같은 이름의 부모 동작을 재정의하고 새 메서드는 자식 고유 행동을 추가합니다. |
| 상속 vs 조합 | 상속은 유형 관계, 조합은 필요한 객체를 부품처럼 포함해 기능을 협력시킵니다. |

## 연결되는 개념

- 공통 클래스와 인스턴스는 [클래스와 인스턴스](02-classes-and-instances.md)에서 설명합니다.
- 부모 private 상태의 경계는 [private 필드와 캡슐화](05-private-fields-and-encapsulation.md)에서 확인할 수 있습니다.
- OOP 네 특성의 전체 그림은 [객체 지향 프로그래밍의 핵심 원리](01-oop-principles.md)에서 이어집니다.

## 셀프 체크

- [ ] 상속 관계를 `extends`로 표현할 수 있다.
- [ ] 파생 생성자에서 `super()` 순서를 설명할 수 있다.
- [ ] 부모 메서드를 재사용해 오버라이딩할 수 있다.

## 복습 질문 및 답변

### Q1. 자식 생성자에서 `this`를 먼저 사용한 뒤 `super()`를 호출할 수 있는가?

<details>
<summary>답</summary>

없습니다. 파생 클래스 생성자에서는 `super()`가 완료된 뒤 `this`를 사용할 수 있습니다.

</details>

### Q2. 오버라이딩한 메서드에서 부모 구현을 호출하는 문법은?

<details>
<summary>답</summary>

`super.methodName()` 형태로 호출합니다.

</details>

### Q3. 상속보다 조합을 고려할 신호는 무엇인가?

<details>
<summary>답</summary>

클래스 사이가 명확한 유형 관계가 아니거나, 여러 기능을 유연하게 교체·결합해야 하거나, 상속 단계가 깊어질 때 조합을 검토할 수 있습니다.

</details>

## 한 줄 정리

> 상속은 공통 기능을 확장하고 `super`와 오버라이딩은 부모 규칙을 보존하면서 자식의 차이를 표현합니다.
