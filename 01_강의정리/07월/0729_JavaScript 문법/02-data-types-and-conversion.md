# JavaScript 데이터 타입과 형 변환

> 같은 모양의 값도 숫자인지 문자열인지에 따라 연산 결과가 달라집니다.

`number` · `string` · `boolean` · `typeof` · `parseInt`

## 핵심요약

- 숫자, 문자열, 불리언은 서로 다른 방식으로 계산되고 비교된다.
- `typeof`는 현재 값의 타입을 확인한다.
- `+`는 숫자를 더하지만 문자열이 포함되면 이어 붙일 수 있다.
- `toString()`은 값을 문자열로, `parseInt()`는 문자열을 정수로 바꾼다.
- 입력값을 계산하기 전에 타입과 변환 시점을 명확히 정해야 한다.

## 1. 기본 데이터 타입

```javascript
var age = 25;
var name = "하늘";
var isMember = true;

console.log(typeof age);      // number
console.log(typeof name);     // string
console.log(typeof isMember); // boolean
```

| 타입 | 예시 | 대표 용도 |
|---|---|---|
| number | `10`, `3.14`, `-2` | 계산, 수량, 점수 |
| string | `"hello"`, `'10'` | 이름, 문장, 식별자 |
| boolean | `true`, `false` | 조건의 참·거짓 |

따옴표로 감싼 `"10"`은 숫자처럼 보여도 문자열입니다.

## 2. 타입에 따라 달라지는 `+`

```javascript
console.log(10 + 2);      // 12
console.log("10" + "2"); // "102"
console.log("10" + 2);   // "102"
```

문자열이 하나라도 포함되면 `+`가 문자열 연결로 동작할 수 있습니다. 계산이 목적이라면 먼저 숫자로 변환해야 합니다.

## 3. 명시적 형 변환

```javascript
var input = "42";
var numberValue = parseInt(input);
var textValue = numberValue.toString();

console.log(numberValue + 8); // 50
console.log(textValue + 8);   // "428"
```

### 변환 선택

| 목적 | 방법 | 결과 예시 |
|---|---|---|
| 문자열을 정수로 | `parseInt("42")` | `42` |
| 값을 문자열로 | `(42).toString()` | `"42"` |
| 타입 확인 | `typeof value` | `"number"` |

`parseInt()` 결과가 숫자로 해석될 수 없으면 `NaN`이 될 수 있으므로 입력을 신뢰하기 전에 확인해야 합니다.

## 코드로 보기 — 자릿수 분리

```javascript
function getDigits(number) {
  var text = number.toString();
  var digits = [];

  for (var i = 0; i < text.length; i++) {
    digits.push(parseInt(text.charAt(i)));
  }

  return digits;
}

console.log(getDigits(1204));
```

### 코드 흐름

1. 숫자를 문자열로 바꿔 각 자리에 접근합니다.
2. `length`만큼 반복합니다.
3. `charAt(i)`로 한 문자를 꺼냅니다.
4. 다시 숫자로 바꾸어 배열에 저장합니다.

### 예상 결과

```text
[1, 2, 0, 4]
```

## 직접 해보기

1. 숫자 `100`과 문자열 `"100"`의 타입을 출력해 보세요.
2. 문자열 `"35"`를 숫자로 바꿔 5를 더해 보세요.
3. 숫자 `2026`을 문자열로 바꾸고 길이를 확인해 보세요.

<details>
<summary>정답 보기</summary>

1. `typeof 100`은 `number`, `typeof "100"`은 `string`입니다.
2. `parseInt("35") + 5`의 결과는 `40`입니다.
3. `(2026).toString().length`의 결과는 `4`입니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| `10` vs `"10"` | 숫자 데이터 vs 문자 데이터 |
| 더하기 vs 이어 붙이기 | 숫자 합산 vs 문자열 결합 |
| `parseInt()` vs `toString()` | 문자열→정수 vs 값→문자열 |
| `NaN` vs 문자열 | 숫자 연산 실패를 나타내는 값 vs 텍스트 |

## 연결되는 개념

- 이전 글: [실행 환경과 변수](01-javascript-runtime-variables.md)
- 다음 글: [프로퍼티와 메서드](03-properties-and-methods.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] number, string, boolean을 구분할 수 있다.
- [ ] `typeof`로 타입을 확인할 수 있다.
- [ ] `+`의 두 역할을 설명할 수 있다.
- [ ] 명시적으로 타입을 변환할 수 있다.

### 복습 질문 및 답변

**Q1. `"2" + 3`이 5가 아닌 이유는 무엇인가요?**
<details><summary>답</summary>문자열이 포함되어 `+`가 연결 연산으로 동작하기 때문입니다.</details>

**Q2. 자릿수에 접근할 때 숫자를 문자열로 바꾸는 이유는 무엇인가요?**
<details><summary>답</summary>문자열은 인덱스로 각 글자에 순서대로 접근할 수 있기 때문입니다.</details>

**Q3. 자동 변환보다 명시적 변환이 좋은 이유는 무엇인가요?**
<details><summary>답</summary>코드 작성자의 의도와 연산 타입이 분명해져 예상 밖 결과를 줄이기 때문입니다.</details>

## 한 줄 정리

> JavaScript 연산을 예측하려면 값의 모양보다 현재 타입을 먼저 확인해야 합니다.
