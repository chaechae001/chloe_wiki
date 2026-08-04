# 구조 분해와 Rest·Spread

> 필요한 값은 이름이나 위치로 꺼내고, 남은 값은 모으고, 묶인 값은 펼쳐서 새로운 데이터를 만듭니다.

`destructuring` · `rest parameter` · `spread syntax` · `default value` · `shallow copy`

## 핵심요약

- 객체 구조 분해는 프로퍼티 이름, 배열 구조 분해는 위치를 기준으로 값을 꺼냅니다.
- 구조 분해에서 기본값과 변수 이름 변경을 함께 사용할 수 있습니다.
- Rest는 여러 값을 하나로 모으고 Spread는 묶인 값을 펼칩니다.
- 나머지 매개변수는 함수 선언의 마지막 매개변수여야 합니다.
- Spread 복사는 중첩 객체까지 깊게 복사하지 않는 얕은 복사입니다.

## 1. 객체와 배열 구조 분해

### 1) 정의

구조 분해 할당은 객체나 배열의 구조를 기준으로 필요한 값을 변수에 연결하는 문법입니다.

### 2) 왜 필요한가

반복되는 `object.property`나 `array[index]` 접근을 줄이고 함수가 실제로 사용하는 데이터가 무엇인지 드러냅니다.

### 3) 핵심 흐름 재구성

```javascript
const user = {
  name: "Mina",
  age: 28,
  contact: { email: "mina@example.com" }
};

const {
  name: displayName,
  grade = "basic",
  contact: { email }
} = user;
```

### 4) 쉬운 예시

여행 가방 전체를 들고 다니는 대신 지금 필요한 여권과 지갑만 꺼내는 과정과 비슷합니다.

### 5) 코드 예시

```javascript
const colors = ["red", "green", "blue"];
const [primary, secondary] = colors;
```

객체는 이름, 배열은 순서가 핵심입니다.

### 6) 헷갈리는 점

기본값은 해당 값이 `undefined`일 때 적용됩니다. 값이 `null`이라면 기본값으로 자동 교체되지 않습니다.

### 7) 한 줄 정리

> 구조 분해는 복합 데이터에서 사용할 부분을 선언부에 명시하는 문법입니다.

## 2. Rest와 Spread

### 1) 정의

Rest는 남은 값을 배열이나 객체로 모으고, Spread는 배열·문자열·객체의 항목을 펼칩니다. 기호는 `...`로 같지만 위치와 방향이 다릅니다.

### 2) 왜 필요한가

인수 개수가 달라지는 함수, 원본을 유지한 복사, 배열 결합, 일부 프로퍼티 변경을 간결하게 표현할 수 있습니다.

### 3) 핵심 흐름 재구성

```javascript
function multiplyAll(factor, ...numbers) {
  return numbers.map((number) => number * factor);
}

console.log(multiplyAll(3, 2, 4, 6)); // [6, 12, 18]
```

`factor`는 첫 인수 하나를 받고 `numbers`는 나머지 인수를 배열로 모읍니다.

### 4) 쉬운 예시

Rest는 흩어진 동전을 주머니에 모으는 동작, Spread는 주머니의 동전을 탁자 위에 하나씩 펼치는 동작과 비슷합니다.

### 5) 코드 예시

```javascript
const baseUser = { name: "Mina", role: "viewer" };
const adminUser = { ...baseUser, role: "admin" };

console.log(baseUser.role);  // viewer
console.log(adminUser.role); // admin
```

뒤에 작성한 프로퍼티가 같은 이름의 앞선 값을 덮어씁니다.

### 6) 헷갈리는 점

```javascript
const original = { profile: { city: "Seoul" } };
const copied = { ...original };
copied.profile.city = "Busan";
```

바깥 객체만 새로 만들어지고 중첩된 `profile`은 같은 객체를 가리키므로 원본의 도시도 바뀝니다.

### 7) 한 줄 정리

> Rest는 여러 값을 받는 쪽에서 모으고, Spread는 보내는 쪽에서 펼칩니다.

## 코드로 보기 — 일부 프로퍼티를 바꾼 복사본

```javascript
function renameUser(user, newName) {
  const { name, ...rest } = user;

  return {
    ...rest,
    name: newName
  };
}

const before = { name: "Mina", age: 28, active: true };
const after = renameUser(before, "Jin");

console.log(before.name); // Mina
console.log(after.name);  // Jin
```

### 코드 목적

원본 객체를 직접 수정하지 않고 특정 프로퍼티만 바꾼 새 객체를 만듭니다.

### 코드 흐름

1. 기존 이름과 나머지 프로퍼티를 분리합니다.
2. 나머지 프로퍼티를 새 객체에 펼칩니다.
3. 새 이름을 추가합니다.
4. 원본과 복사본을 비교합니다.

### 실행 결과 해석

새 객체의 이름만 바뀌며 원본 객체의 이름은 유지됩니다.

### 실무 연결

React 상태 업데이트, API 요청 데이터 조립, 설정 객체 병합처럼 원본을 유지해야 하는 작업에 자주 쓰입니다.

## 직접 해보기

1. 배열 `[10, 20, 30]`의 첫 값과 나머지를 구조 분해해 보세요.
2. 함수의 첫 인수와 나머지 인수를 분리하는 매개변수를 작성해 보세요.
3. 사용자 객체의 `active`만 `false`로 바꾼 복사본을 Spread로 만들어 보세요.

<details>
<summary>정답 보기</summary>

1. `const [first, ...rest] = [10, 20, 30];`처럼 작성합니다.
2. `function example(first, ...rest) {}`처럼 Rest 매개변수를 마지막에 둡니다.
3. `const updated = { ...user, active: false };`처럼 뒤에서 덮어씁니다.

</details>

## 헷갈리기 쉬운 포인트

| 헷갈리는 개념 | 차이 |
|---|---|
| 객체 구조 분해 vs 배열 구조 분해 | 프로퍼티 이름을 기준으로 꺼내는지 위치를 기준으로 꺼내는지 다릅니다. |
| Rest vs Spread | 값을 모으는지 기존 묶음을 펼치는지 다릅니다. |
| 얕은 복사 vs 깊은 복사 | 중첩 참조를 공유하는지 내부까지 새로 복사하는지 다릅니다. |

## 연결되는 개념

- 이전에 알면 좋은 개념: [화살표 함수와 간결한 표현](02-arrow-functions-and-modern-expressions.md)
- 다음에 이어지는 개념: [고차 함수와 콜백](04-higher-order-functions-and-callbacks.md)
- 함께 보면 좋은 키워드: `불변성`, `참조 타입`, `기본 매개변수`

## 셀프 체크

- [ ] 객체와 배열 구조 분해의 기준을 구분할 수 있다.
- [ ] 구조 분해에서 기본값과 이름 변경을 사용할 수 있다.
- [ ] Rest와 Spread의 방향을 설명할 수 있다.
- [ ] 나머지 매개변수의 위치 규칙을 안다.
- [ ] Spread의 얕은 복사 한계를 설명할 수 있다.

### 복습 질문 및 답변

**Q1. 배열 구조 분해에서 변수 이름은 원소 이름과 같아야 하나요?**

<details>
<summary>답</summary>

아닙니다. 배열은 위치 순서로 연결되므로 변수 이름은 자유롭게 정할 수 있습니다.

</details>

**Q2. Rest 매개변수는 어떤 자료형으로 값을 모으나요?**

<details>
<summary>답</summary>

함수에 전달된 나머지 인수를 배열로 모읍니다.

</details>

**Q3. `{ ...object }`만으로 중첩 객체까지 독립적으로 복사되나요?**

<details>
<summary>답</summary>

아닙니다. 바깥 객체만 새로 만들고 중첩된 참조값은 원본과 공유하는 얕은 복사입니다.

</details>

## 한 줄 정리

> 구조 분해와 Rest·Spread를 함께 사용하면 복합 데이터를 꺼내고 조립하는 흐름을 선언적으로 표현할 수 있습니다.
