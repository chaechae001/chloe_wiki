# 반복문과 순차 탐색

> 반복문은 같은 코드를 줄이는 문법이 아니라, 데이터의 모든 항목을 빠짐없이 검사하는 구조입니다.

`for` · `while` · `인덱스` · `누적` · `무한 반복`

## 핵심요약

- `for`문은 초기값, 조건식, 증감식으로 반복 범위를 표현한다.
- 배열과 문자열은 인덱스를 이용해 순서대로 탐색한다.
- 합계나 결과 배열은 반복문 밖에서 초기화한다.
- 조건식과 증감식이 맞지 않으면 무한 반복이나 범위 오류가 생긴다.
- 현재 인덱스와 중간 결과를 출력하면 반복 오류를 쉽게 찾을 수 있다.

## 1. `for`문의 세 부분

```javascript
for (var i = 0; i < 5; i++) {
  console.log(i);
}
```

| 부분 | 역할 | 실행 시점 |
|---|---|---|
| `var i = 0` | 시작 위치 | 처음 한 번 |
| `i < 5` | 계속할 조건 | 매 반복 전 |
| `i++` | 다음 위치로 이동 | 매 반복 후 |

출력은 `0, 1, 2, 3, 4`이며 조건이 거짓이 되는 순간 반복을 멈춥니다.

## 2. 배열 순회와 누적

```javascript
var scores = [80, 90, 70];
var total = 0;

for (var i = 0; i < scores.length; i++) {
  total = total + scores[i];
}

console.log(total);
```

누적 변수 `total`을 반복문 안에서 0으로 초기화하면 매번 이전 계산을 잃습니다. 결과를 계속 쌓을 값은 반복문 밖에서 만듭니다.

## 3. `while`문

```javascript
var count = 3;

while (count > 0) {
  console.log(count);
  count--;
}
```

반복 횟수가 분명하면 `for`가 읽기 쉽고, 특정 상태가 될 때까지 반복한다면 `while`이 자연스럽습니다.

## 코드로 보기 — 조건에 맞는 이름 모으기

```javascript
function getAdultNames(people) {
  var names = [];

  for (var i = 0; i < people.length; i++) {
    if (people[i].age >= 20) {
      names.push(people[i].name);
    }
  }

  return names;
}

var people = [
  { name: "도윤", age: 24 },
  { name: "서아", age: 17 },
  { name: "지후", age: 20 }
];

console.log(getAdultNames(people));
```

### 예상 결과

```text
["도윤", "지후"]
```

### 문제 해결 패턴

1. 빈 결과 배열을 만듭니다.
2. 모든 요소를 순회합니다.
3. 현재 요소가 조건을 만족하는지 검사합니다.
4. 필요한 값만 결과에 추가합니다.
5. 반복이 끝난 뒤 결과를 반환합니다.

## 직접 해보기

1. 1부터 10까지의 합을 반복문으로 구해 보세요.
2. 배열에서 3의 배수만 새 배열에 모아 보세요.
3. 문자열의 각 글자를 한 줄씩 출력해 보세요.

<details>
<summary>정답 보기</summary>

1. `total = 0`을 밖에 두고 `i = 1`부터 10까지 더합니다.
2. `numbers[i] % 3 === 0`일 때 `push()`합니다.
3. `i < text.length` 동안 `text.charAt(i)`를 출력합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| `i < length` vs `i <= length` | 유효 인덱스까지만 vs 범위 밖까지 접근 |
| 초기화 위치 | 반복 밖이면 누적, 안이면 매번 초기화 |
| `for` vs `while` | 범위 중심 vs 상태 중심 |
| 현재 요소 vs 인덱스 | `items[i]` vs `i` |

## 연결되는 개념

- 이전 글: [연산자와 조건문](04-operators-and-conditionals.md)
- 다음 글: [함수와 구조화된 데이터](06-functions-and-structured-data.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] `for`문의 세 부분을 설명할 수 있다.
- [ ] 배열 전체를 안전하게 순회할 수 있다.
- [ ] 합계와 결과 배열을 누적할 수 있다.
- [ ] 무한 반복과 범위 오류를 점검할 수 있다.

### 복습 질문 및 답변

**Q1. 배열 순회 조건에 `<`를 사용하는 이유는 무엇인가요?**
<details><summary>답</summary>마지막 인덱스가 길이보다 1 작기 때문입니다.</details>

**Q2. 결과 배열은 왜 반복문 밖에서 만드나요?**
<details><summary>답</summary>각 반복에서 추가한 값을 다음 반복에도 유지하기 위해서입니다.</details>

**Q3. 반복이 끝나지 않으면 무엇을 먼저 확인하나요?**
<details><summary>답</summary>조건이 언젠가 거짓이 되는지, 반복 변수가 올바른 방향으로 바뀌는지 확인합니다.</details>

## 한 줄 정리

> 반복문은 시작·종료·변화 규칙을 명확히 하고 중간 결과를 추적할 때 안전합니다.
