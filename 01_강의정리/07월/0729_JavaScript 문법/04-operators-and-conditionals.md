# 연산자와 조건문

> 조건문은 계산한 결과를 바탕으로 프로그램이 어느 길로 갈지 결정합니다.

`산술 연산자` · `비교 연산자` · `논리 연산자` · `if` · `else`

## 핵심요약

- 산술 연산자는 값을 계산하고 비교 연산자는 참·거짓을 만든다.
- `===`와 `!==`는 값과 타입을 함께 비교한다.
- `&&`, `||`, `!`로 여러 조건을 조합한다.
- `if`, `else if`, `else`는 조건에 따라 실행 경로를 나눈다.
- 복잡한 조건은 의미 있는 불리언 변수로 나누면 읽기 쉽다.

## 1. 산술·증감 연산자

```javascript
var score = 10;

score = score + 5;
score++;
score--;

console.log(score);
```

| 연산자 | 의미 | 예시 결과 |
|---|---|---|
| `+ - * /` | 기본 계산 | `6 / 2` → `3` |
| `%` | 나머지 | `7 % 2` → `1` |
| `++` | 1 증가 | `count++` |
| `--` | 1 감소 | `count--` |

나머지 연산자는 홀짝 판단이나 일정한 주기의 구분에 자주 사용합니다.

## 2. 비교·논리 연산자

```javascript
var age = 22;
var hasTicket = true;

var canEnter = age >= 20 && hasTicket === true;
console.log(canEnter);
```

| 연산자 | 질문 |
|---|---|
| `===` | 값과 타입이 모두 같은가? |
| `!==` | 값 또는 타입이 다른가? |
| `>`, `>=` | 왼쪽이 더 큰가, 같은 것도 포함하는가? |
| `&&` | 두 조건이 모두 참인가? |
| `||` | 하나 이상 참인가? |
| `!` | 참·거짓을 뒤집는가? |

## 3. 조건문

```javascript
var temperature = 28;

if (temperature >= 30) {
  console.log("매우 더움");
} else if (temperature >= 20) {
  console.log("따뜻함");
} else {
  console.log("쌀쌀함");
}
```

위에서부터 조건을 검사하고 처음 참이 된 블록만 실행합니다. 따라서 더 구체적이거나 큰 범위의 조건을 먼저 두어야 합니다.

## 코드로 보기 — 간단한 입력 검증

```javascript
function isBasicEmail(email) {
  var hasAt = email.indexOf("@") !== -1;
  var hasDot = email.indexOf(".") !== -1;
  var isLongEnough = email.length >= 5;

  if (hasAt && hasDot && isLongEnough) {
    return true;
  }

  return false;
}

console.log(isBasicEmail("user@example.com"));
```

이 코드는 학습용 최소 조건만 검사합니다. 실제 서비스의 이메일 검증 규칙 전체를 대체하지는 않습니다.

### 예상 결과

```text
true
```

## 직접 해보기

1. 숫자가 짝수인지 `%`와 `===`로 확인해 보세요.
2. 점수가 90 이상이면 A, 80 이상이면 B, 나머지는 C를 출력해 보세요.
3. 나이 20 이상이면서 동의 여부가 참일 때만 통과시키는 조건을 작성해 보세요.

<details>
<summary>정답 보기</summary>

1. `number % 2 === 0`으로 판단합니다.
2. 큰 경계값부터 `if`, `else if`, `else` 순으로 작성합니다.
3. `age >= 20 && agreed === true`처럼 두 조건을 `&&`로 묶습니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| `=` vs `===` | 값 저장 vs 값·타입 비교 |
| `==` vs `===` | 타입 변환 후 비교 vs 타입까지 엄격 비교 |
| `&&` vs `||` | 모두 참 vs 하나 이상 참 |
| `if` 연속 vs `else if` | 각각 검사 vs 첫 참 이후 중단 |

## 연결되는 개념

- 이전 글: [프로퍼티와 메서드](03-properties-and-methods.md)
- 다음 글: [반복문](05-loops-and-iteration.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 산술·비교·논리 연산자를 구분할 수 있다.
- [ ] 엄격 비교를 사용할 수 있다.
- [ ] 경계값을 고려해 조건 순서를 정할 수 있다.
- [ ] 복합 조건을 불리언 변수로 나눌 수 있다.

### 복습 질문 및 답변

**Q1. `10 === "10"`의 결과는 무엇인가요?**
<details><summary>답</summary>`false`입니다. 값의 모양은 같아도 타입이 다릅니다.</details>

**Q2. `else if` 순서가 중요한 이유는 무엇인가요?**
<details><summary>답</summary>처음 참인 블록만 실행되므로 넓은 조건이 앞에 오면 뒤의 세부 조건에 도달하지 못할 수 있습니다.</details>

**Q3. 조건을 변수로 나누면 무엇이 좋아지나요?**
<details><summary>답</summary>각 판단의 의미를 이름으로 읽고 개별 결과를 출력해 디버깅할 수 있습니다.</details>

## 한 줄 정리

> 조건문은 비교 결과를 실행 경로로 바꾸며, 좋은 조건식은 경계와 의도가 분명합니다.
