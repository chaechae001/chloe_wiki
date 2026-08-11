# 접근자 프로퍼티와 상태 검증

getter와 setter는 메서드를 프로퍼티처럼 읽고 쓰게 하면서, 조회 형식과 변경 규칙을 한곳에 모으는 문법입니다.

## 핵심 키워드

`getter` · `setter` · `accessor property` · `validation` · `read-only`

## 핵심 요약

- getter는 값을 계산하거나 가공해 프로퍼티처럼 제공합니다.
- setter는 새 값을 받기 전에 검증하거나 변환할 수 있습니다.
- 접근자에는 호출 괄호를 붙이지 않습니다.
- getter와 setter만으로 저장 프로퍼티가 완전히 숨겨지는 것은 아닙니다.

## 1. 접근자 프로퍼티

접근자 프로퍼티는 내부적으로 함수이지만 사용하는 쪽에서는 일반 프로퍼티처럼 보입니다.

```javascript
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }

  get fahrenheit() {
    return this._celsius * 1.8 + 32;
  }

  set celsius(value) {
    if (!Number.isFinite(value)) {
      throw new TypeError("온도는 숫자여야 합니다.");
    }
    this._celsius = value;
  }

  get celsius() {
    return this._celsius;
  }
}

const temperature = new Temperature(20);
console.log(temperature.fahrenheit); // 68
temperature.celsius = 25;
```

`temperature.fahrenheit()`가 아니라 `temperature.fahrenheit`로 읽고, setter도 대입 문법으로 호출합니다.

## 2. 조회 가공과 변경 검증

getter는 내부 표현을 그대로 노출하지 않고 사용하기 좋은 형태로 만들 수 있습니다. setter는 객체 규칙을 깨뜨리는 값을 거절하거나 정규화합니다.

| 목적 | getter | setter |
|---|---|---|
| 값 읽기 | 계산·조합·형식화 | 해당 없음 |
| 값 변경 | 해당 없음 | 검증·변환 후 저장 |
| 사용 문법 | `object.value` | `object.value = next` |

계산이 매우 비싸거나 부작용이 있는 작업은 프로퍼티 접근만으로 비용을 예상하기 어렵습니다. 그런 경우 일반 메서드가 더 명확할 수 있습니다.

## 3. 자기 호출 문제 피하기

setter 안에서 같은 접근자에 다시 대입하면 무한 재귀가 생깁니다.

```javascript
class Score {
  constructor(value) {
    this.value = value;
  }

  set value(nextValue) {
    if (nextValue < 0 || nextValue > 100) {
      throw new RangeError("점수는 0부터 100 사이여야 합니다.");
    }
    this._value = nextValue;
  }

  get value() {
    return this._value;
  }
}
```

setter `value` 안에서는 `this.value = ...`가 아니라 다른 저장 공간인 `this._value`를 사용합니다. 밑줄은 관례일 뿐 실제 접근 제한은 아닙니다.

## 대표 코드: 주문 수량 통제

### 목적

수량을 읽고 바꾸는 인터페이스에 정수와 범위 검증을 적용합니다.

```javascript
class OrderItem {
  constructor(name, quantity) {
    this.name = name;
    this.quantity = quantity;
  }

  set quantity(value) {
    if (!Number.isInteger(value) || value < 1) {
      throw new RangeError("수량은 1 이상의 정수여야 합니다.");
    }
    this._quantity = value;
  }

  get quantity() {
    return this._quantity;
  }

  get label() {
    return `${this.name} × ${this.quantity}`;
  }
}

const item = new OrderItem("노트", 2);
item.quantity = 3;
console.log(item.label); // 노트 × 3
```

### 코드 흐름과 결과

1. 생성자에서도 setter를 거쳐 같은 검증 규칙을 재사용합니다.
2. 잘못된 수량은 저장되기 전에 거절됩니다.
3. getter `label`이 여러 상태를 표시 문자열로 조합합니다.
4. 호출자는 검증 로직의 위치를 몰라도 안전한 인터페이스를 사용합니다.

### 실무 연결

가격, 수량, 상태 코드처럼 값 범위와 형식이 중요한 도메인 프로퍼티의 경계를 통제할 때 유용합니다.

## 직접 해보기

1. getter와 일반 프로퍼티의 사용 문법이 같은 이유를 설명하세요.
2. 최대 수량을 99로 제한하도록 setter를 수정하세요.
3. getter에서 네트워크 요청을 수행하는 설계가 왜 혼란스러울 수 있는지 설명하세요.

<details>
<summary>정답 보기</summary>

1. 접근자 프로퍼티는 함수 동작을 프로퍼티 읽기·쓰기 인터페이스로 제공하기 때문입니다.
2. 조건에 `value > 99`를 추가하고 범위 오류를 던집니다.
3. 단순 값 읽기처럼 보이지만 느린 비동기 작업과 실패 가능성이 숨어 비용을 예측하기 어렵습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| getter vs 일반 메서드 | getter는 프로퍼티처럼 읽고 일반 메서드는 괄호로 명시적으로 호출합니다. |
| setter vs 직접 대입 | setter는 검증과 변환을 거치지만 공개 필드 직접 대입은 값을 바로 바꿉니다. |
| `_value` vs `#value` | 밑줄은 관례이고 `#` private 필드는 언어 수준에서 외부 접근을 제한합니다. |

## 연결되는 개념

- 초기값 저장은 [생성자와 this](03-constructor-and-this.md)에서 설명합니다.
- 실제 비공개 상태는 [private 필드와 캡슐화](05-private-fields-and-encapsulation.md)에서 이어집니다.
- 상속받은 접근자 활용은 [상속과 super, 오버라이딩](06-inheritance-super-and-overriding.md)에서 다룹니다.

## 셀프 체크

- [ ] getter와 setter의 호출 문법을 알고 있다.
- [ ] setter로 값 검증을 구현할 수 있다.
- [ ] 같은 접근자를 재호출하는 문제를 피할 수 있다.

## 복습 질문 및 답변

### Q1. getter를 읽을 때 괄호를 붙이는가?

<details>
<summary>답</summary>

붙이지 않습니다. 일반 프로퍼티를 읽는 것처럼 `object.value`로 사용합니다.

</details>

### Q2. setter 안에서 `this.value = next`를 실행하면 어떤 문제가 생길 수 있는가?

<details>
<summary>답</summary>

같은 setter가 계속 자신을 호출해 무한 재귀와 스택 오류가 발생할 수 있습니다.

</details>

### Q3. getter만 있고 setter가 없으면 어떤 인터페이스가 되는가?

<details>
<summary>답</summary>

외부에서 값을 읽을 수 있지만 해당 접근자를 통한 대입은 허용되지 않는 읽기 전용 형태가 됩니다.

</details>

## 한 줄 정리

> 접근자는 프로퍼티처럼 자연스러운 사용법을 유지하면서 조회 가공과 변경 검증을 객체 경계에 모읍니다.
