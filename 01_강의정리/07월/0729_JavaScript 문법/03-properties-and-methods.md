# 문자열과 배열의 프로퍼티·메서드

> 값은 데이터만 담는 것이 아니라, 그 데이터를 다루는 기능도 함께 제공합니다.

`length` · `charAt()` · `indexOf()` · `push()` · `pop()`

## 핵심요약

- 프로퍼티는 값이 가진 정보이고 메서드는 값에 실행하는 기능이다.
- 문자열과 배열의 `length`는 길이를 알려 준다.
- 문자열은 `charAt()`과 `indexOf()`로 문자를 찾을 수 있다.
- 배열은 `push()`와 `pop()`으로 끝 요소를 추가하거나 제거한다.
- 인덱스는 0부터 시작하므로 마지막 위치는 `length - 1`이다.

## 1. 프로퍼티와 메서드

```javascript
var message = "JavaScript";

console.log(message.length);
console.log(message.charAt(0));
console.log(message.indexOf("Script"));
```

| 표현 | 종류 | 의미 |
|---|---|---|
| `message.length` | 프로퍼티 | 문자열 길이 |
| `message.charAt(0)` | 메서드 | 0번 위치의 문자 |
| `message.indexOf("S")` | 메서드 | 첫 일치 위치 |

메서드는 실행하므로 괄호가 필요하지만 `length`는 값의 정보이므로 괄호를 쓰지 않습니다.

## 2. 인덱스와 길이

```javascript
var colors = ["red", "green", "blue"];

console.log(colors[0]);
console.log(colors[colors.length - 1]);
```

세 요소의 인덱스는 `0`, `1`, `2`입니다. 길이는 3이므로 마지막 인덱스는 `3 - 1`입니다.

## 3. 배열 변경 메서드

```javascript
var tasks = ["읽기", "정리"];

tasks.push("복습");
console.log(tasks);

var removed = tasks.pop();
console.log(removed);
console.log(tasks);
```

`push()`는 배열 끝에 값을 추가하고 새 길이를 반환합니다. `pop()`은 마지막 값을 제거하고 그 값을 반환합니다.

## 코드로 보기 — 홀수만 모으기

```javascript
function getOddNumbers(numbers) {
  var result = [];

  for (var i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 !== 0) {
      result.push(numbers[i]);
    }
  }

  return result;
}

console.log(getOddNumbers([1, 2, 3, 4, 5]));
```

### 코드 흐름

1. 결과를 담을 빈 배열을 만듭니다.
2. 입력 배열의 길이만큼 반복합니다.
3. 현재 요소를 2로 나눈 나머지를 확인합니다.
4. 홀수만 `push()`로 결과 배열에 추가합니다.

### 예상 결과

```text
[1, 3, 5]
```

## 직접 해보기

1. `"frontend"`의 첫 글자와 마지막 글자를 출력해 보세요.
2. 배열 끝에 두 값을 추가한 뒤 마지막 값을 제거해 보세요.
3. 문자열에 `"@"`가 포함됐는지 `indexOf()`로 확인해 보세요.

<details>
<summary>정답 보기</summary>

1. `text.charAt(0)`과 `text.charAt(text.length - 1)`을 사용합니다.
2. `push()`를 두 번 실행하고 `pop()`을 한 번 실행합니다.
3. `email.indexOf("@") !== -1`이면 포함된 상태입니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 길이 vs 마지막 인덱스 | `length` vs `length - 1` |
| 프로퍼티 vs 메서드 | 정보 읽기 vs 기능 실행 |
| `push()` vs `pop()` | 끝에 추가 vs 끝에서 제거 |
| `indexOf()`의 `-1` | 찾지 못했다는 뜻 |

## 연결되는 개념

- 이전 글: [데이터 타입과 형 변환](02-data-types-and-conversion.md)
- 다음 글: [연산자와 조건문](04-operators-and-conditionals.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 프로퍼티와 메서드를 구분할 수 있다.
- [ ] 0부터 시작하는 인덱스를 사용할 수 있다.
- [ ] 문자열에서 문자를 찾을 수 있다.
- [ ] 배열 끝 요소를 추가하고 제거할 수 있다.

### 복습 질문 및 답변

**Q1. 요소가 5개인 배열의 마지막 인덱스는 무엇인가요?**
<details><summary>답</summary>`4`입니다. 인덱스는 0부터 시작합니다.</details>

**Q2. `indexOf()`가 0을 반환하면 찾지 못한 것인가요?**
<details><summary>답</summary>아닙니다. 검색값이 첫 위치에 있다는 뜻입니다.</details>

**Q3. 결과 배열을 따로 만드는 이유는 무엇인가요?**
<details><summary>답</summary>입력은 유지하면서 조건을 만족한 값만 새로운 결과로 반환하기 위해서입니다.</details>

## 한 줄 정리

> 프로퍼티로 상태를 읽고 메서드로 값을 다루면 문자열과 배열 문제를 작은 단계로 풀 수 있습니다.
