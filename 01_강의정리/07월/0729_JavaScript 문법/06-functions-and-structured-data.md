# 함수와 배열·객체 데이터

> 함수는 입력을 받아 규칙을 적용하고 결과를 돌려주는 재사용 가능한 문제 해결 단위입니다.

`function` · `parameter` · `argument` · `return` · `object`

## 핵심요약

- 함수는 반복되는 로직에 이름을 붙여 한곳에서 관리한다.
- 매개변수는 입력 자리이고 인자는 호출할 때 전달하는 실제 값이다.
- `return`은 결과를 호출한 위치로 돌려주고 함수를 종료한다.
- 배열은 순서가 있는 목록, 객체는 이름이 있는 속성 묶음이다.
- 복합 데이터는 반복문과 조건문을 결합해 원하는 결과로 가공한다.

## 1. 함수 정의와 호출

```javascript
function add(a, b) {
  return a + b;
}

var result = add(3, 5);
console.log(result);
```

| 용어 | 예시 | 의미 |
|---|---|---|
| 함수명 | `add` | 기능을 나타내는 이름 |
| 매개변수 | `a`, `b` | 입력을 받을 자리 |
| 인자 | `3`, `5` | 호출 시 전달한 실제 값 |
| 반환값 | `8` | 호출 결과 |

## 2. `return`의 역할

```javascript
function isEven(number) {
  return number % 2 === 0;
}

if (isEven(8)) {
  console.log("짝수");
}
```

`console.log()`는 화면 확인용이고 `return`은 결과를 다른 코드에서 사용하게 합니다. 반환 이후의 문장은 실행되지 않습니다.

## 3. 배열과 객체

```javascript
var user = {
  name: "하람",
  age: 21
};

var users = [
  user,
  { name: "나래", age: 18 }
];

console.log(users[0].name);
```

배열은 인덱스로, 객체는 속성 이름으로 접근합니다. 객체 배열에서는 `users[i].age`처럼 두 접근 방식을 연결합니다.

## 코드로 보기 — 직각삼각형 판별

```javascript
function isRightTriangle(hypotenuse, sideA, sideB) {
  var hypotenuseSquared = hypotenuse * hypotenuse;
  var otherSidesSquared = sideA * sideA + sideB * sideB;

  return hypotenuseSquared === otherSidesSquared;
}

console.log(isRightTriangle(5, 3, 4));
console.log(isRightTriangle(10, 3, 2));
```

### 예상 결과

```text
true
false
```

### 설계 포인트

1. 입력의 의미가 드러나는 매개변수 이름을 정합니다.
2. 비교할 두 값을 각각 계산합니다.
3. 엄격 비교 결과를 바로 반환합니다.
4. 대표 입력과 경계 입력으로 확인합니다.

## 직접 해보기

1. 두 수 중 큰 값을 반환하는 함수를 작성해 보세요.
2. 숫자 배열의 합계를 반환하는 함수를 작성해 보세요.
3. 객체 배열에서 20세 미만 이름만 반환해 보세요.

<details>
<summary>정답 보기</summary>

1. 두 값을 비교해 큰 값을 `return`합니다.
2. 합계 변수를 0으로 두고 배열 전체를 순회한 뒤 반환합니다.
3. 빈 배열을 만들고 `age < 20`인 객체의 `name`만 추가합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 매개변수 vs 인자 | 함수 정의의 입력 자리 vs 호출 시 실제 값 |
| 출력 vs 반환 | 콘솔 표시 vs 다음 코드에 결과 전달 |
| 배열 vs 객체 | 순서 기반 목록 vs 이름 기반 속성 묶음 |
| `items[i]` vs `item.name` | 배열 요소 접근 vs 객체 속성 접근 |

## 연결되는 개념

- 이전 글: [반복문](05-loops-and-iteration.md)
- 다음 글: [문제 해결 워크숍](07-javascript-problem-solving-workshop.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 함수를 정의하고 호출할 수 있다.
- [ ] 매개변수와 인자를 구분할 수 있다.
- [ ] 결과를 반환해 다른 코드에서 사용할 수 있다.
- [ ] 객체 배열의 속성을 순회할 수 있다.

### 복습 질문 및 답변

**Q1. 함수 안에서 계산만 하고 반환하지 않으면 어떻게 되나요?**
<details><summary>답</summary>호출 결과는 기본적으로 `undefined`가 되어 외부 코드가 계산값을 사용할 수 없습니다.</details>

**Q2. 객체 배열의 두 번째 사람 이름은 어떻게 읽나요?**
<details><summary>답</summary>`people[1].name`처럼 배열 인덱스와 객체 속성을 연결합니다.</details>

**Q3. 조건식 결과를 바로 반환해도 되나요?**
<details><summary>답</summary>네. 조건식 자체가 boolean을 만들면 `return condition;`으로 간결하게 작성할 수 있습니다.</details>

## 한 줄 정리

> 함수는 입력·처리·반환의 경계를 만들고, 배열과 객체는 현실의 여러 데이터를 구조화합니다.
