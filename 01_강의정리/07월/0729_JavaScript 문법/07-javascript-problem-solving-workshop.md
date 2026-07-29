# JavaScript 문제 해결 워크숍

> 문법을 아는 것과 문제를 푸는 것의 차이는 요구사항을 작은 단계로 바꾸는 능력에서 생깁니다.

`요구사항 분석` · `입출력` · `분해` · `경계값` · `검증`

## 핵심요약

- 문제를 함수명, 입력, 출력, 조건으로 나누어 읽는다.
- 예시를 손으로 계산해 변환 과정을 먼저 확인한다.
- 변수·조건문·반복문을 한 단계씩 조합한다.
- 완성 예시를 외우기보다 중간값과 경계값으로 자신의 로직을 검증한다.
- 정상 입력뿐 아니라 빈 값, 경계값, 타입 차이도 점검한다.

## 1. 요구사항을 코드 구조로 번역하기

예를 들어 “숫자 배열에서 홀수만 반환”이라는 요구사항은 다음처럼 분해됩니다.

| 질문 | 결정 |
|---|---|
| 입력은 무엇인가? | 숫자 배열 한 개 |
| 출력은 무엇인가? | 홀수만 담긴 새 배열 |
| 모든 값을 봐야 하는가? | 반복문 필요 |
| 어떤 조건인가? | `value % 2 !== 0` |
| 결과를 어떻게 모으는가? | 빈 배열과 `push()` |

이 표가 코드의 뼈대가 됩니다.

## 2. 단계별 구현

```javascript
function getOddNumbers(numbers) {
  var result = [];

  for (var i = 0; i < numbers.length; i++) {
    var current = numbers[i];

    if (current % 2 !== 0) {
      result.push(current);
    }
  }

  return result;
}
```

처음부터 한 줄로 줄이기보다 `current`처럼 중간값에 이름을 붙이면 각 단계의 의미를 확인하기 쉽습니다.

## 3. 대표값과 경계값

```javascript
console.log(getOddNumbers([1, 2, 3])); // [1, 3]
console.log(getOddNumbers([2, 4]));    // []
console.log(getOddNumbers([]));        // []
console.log(getOddNumbers([-3, 0]));   // [-3]
```

| 테스트 | 확인 목적 |
|---|---|
| 홀수와 짝수 혼합 | 기본 로직 |
| 조건 만족값 없음 | 빈 결과 처리 |
| 빈 배열 | 반복 0회 처리 |
| 음수·0 | 연산 경계 이해 |

## 4. 자릿수 문제 비교

숫자를 자릿수 배열로 바꿀 때는 숫자 → 문자열 → 문자 순회 → 숫자 변환의 흐름을 사용합니다. 반대 문제는 배열 순회 → 문자열 연결 → 숫자 변환의 흐름입니다.

```javascript
function makeNumberFromDigits(digits) {
  var text = "";

  for (var i = 0; i < digits.length; i++) {
    text = text + digits[i].toString();
  }

  return parseInt(text);
}
```

선행 0이 있는 배열은 숫자로 바꾸는 순간 0이 사라질 수 있습니다. 출력의 의미가 “숫자”인지 “식별 문자열”인지 요구사항을 먼저 확인해야 합니다.

## 코드로 보기 — 객체 배열 필터링

```javascript
function getNamesByAge(people, minimumAge) {
  var names = [];

  for (var i = 0; i < people.length; i++) {
    var person = people[i];

    if (person.age >= minimumAge) {
      names.push(person.name);
    }
  }

  return names;
}

var people = [
  { name: "유진", age: 19 },
  { name: "준호", age: 20 },
  { name: "소미", age: 31 }
];

console.log(getNamesByAge(people, 20));
```

### 예상 결과

```text
["준호", "소미"]
```

## 직접 해보기

1. 문자열에 `@`, `.`, 최소 길이 조건이 모두 있는지 검사하는 함수를 설계해 보세요.
2. 배열의 2·5·7번째 값을 더할 때 필요한 인덱스를 적어 보세요.
3. 세 변으로 직각삼각형을 판별하는 함수의 테스트 사례를 세 개 만드세요.

<details>
<summary>정답 보기</summary>

1. 세 조건을 불리언 변수로 나누고 `&&`로 결합합니다. 이는 학습용 최소 검증임도 구분합니다.
2. 인덱스는 0부터 시작하므로 `1`, `4`, `6`입니다.
3. 참 사례, 거짓 사례, 경계나 예상 밖 입력을 각각 포함합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 문제 예시 vs 일반 규칙 | 한 입력의 결과 vs 모든 입력에 적용할 로직 |
| 인덱스 vs 순번 | 0부터 시작 vs 1부터 세는 표현 |
| 확인용 출력 vs 최종 반환 | 디버깅 정보 vs 함수 계약 |
| 정답 암기 vs 패턴 학습 | 특정 코드 복제 vs 입력·조건·누적 구조 이해 |

## 연결되는 개념

- 이전 글: [함수와 구조화된 데이터](06-functions-and-structured-data.md)
- 용어 확인: [GLOSSARY](GLOSSARY.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] 요구사항에서 입력과 출력을 찾을 수 있다.
- [ ] 반복·조건·누적 패턴을 선택할 수 있다.
- [ ] 예시를 손으로 추적할 수 있다.
- [ ] 정상값과 경계값을 나누어 테스트할 수 있다.
- [ ] 정답 없이도 중간값을 출력해 오류를 찾을 수 있다.

### 복습 질문 및 답변

**Q1. 코드를 쓰기 전에 입출력을 적어야 하는 이유는 무엇인가요?**
<details><summary>답</summary>함수가 받아야 할 값과 반드시 돌려줘야 할 결과가 명확해져 불필요한 구현을 줄이기 때문입니다.</details>

**Q2. 빈 배열 테스트가 중요한 이유는 무엇인가요?**
<details><summary>답</summary>반복이 한 번도 실행되지 않을 때 초기값과 반환 구조가 올바른지 확인할 수 있기 때문입니다.</details>

**Q3. 중간 변수를 쓰는 장점은 무엇인가요?**
<details><summary>답</summary>복잡한 식을 의미 있는 단계로 나누고 각 값을 따로 확인할 수 있습니다.</details>

## 한 줄 정리

> 문제 해결은 긴 정답을 떠올리는 일이 아니라 입력을 작은 판단과 반복으로 바꾸고 검증하는 과정입니다.
