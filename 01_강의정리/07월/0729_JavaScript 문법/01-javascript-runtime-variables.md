# JavaScript 실행 환경과 변수

> JavaScript 학습의 출발점은 값을 저장하고, 실행 결과를 직접 확인하는 것입니다.

`script` · `console` · `var` · `선언` · `재할당`

## 핵심요약

- JavaScript는 HTML의 `script` 태그로 연결하거나 브라우저 콘솔에서 실행할 수 있다.
- 변수는 값에 이름을 붙여 저장하고 다시 사용하는 공간이다.
- 선언은 공간을 만들고, 초기화는 첫 값을 저장하며, 재할당은 값을 바꾼다.
- 변수 이름은 목적을 드러내야 코드의 흐름을 읽기 쉽다.
- 실행 결과와 현재 값은 `console.log()`로 확인한다.

## 1. HTML과 JavaScript 연결

### 정의

JavaScript 파일은 HTML 문서의 `script` 태그를 통해 브라우저에 연결됩니다.

```html
<body>
  <h1>오늘의 메시지</h1>
  <script src="index.js"></script>
</body>
```

### 실행 순서

1. 브라우저가 HTML을 위에서 아래로 읽습니다.
2. `script` 태그를 만나면 지정한 파일을 불러옵니다.
3. JavaScript 문장을 순서대로 실행합니다.
4. 오류가 나면 개발자 도구의 Console에서 위치와 원인을 확인합니다.

`script`를 `body` 끝에 두면 앞의 화면 요소가 먼저 만들어진 뒤 코드가 실행되어 초보자가 흐름을 이해하기 쉽습니다.

## 2. 선언·초기화·재할당

### 세 동작 구분

```javascript
var fruit;          // 선언
fruit = "apple";   // 초기화
fruit = "banana";  // 재할당
```

| 동작 | 의미 | 예시 |
|---|---|---|
| 선언 | 이름이 있는 저장 공간 생성 | `var score;` |
| 초기화 | 처음 값을 저장 | `score = 80;` |
| 선언과 초기화 | 두 동작을 한 번에 수행 | `var score = 80;` |
| 재할당 | 기존 변수의 값을 변경 | `score = 90;` |

재할당할 때는 `var`를 다시 쓰지 않습니다. 이미 만들어진 공간에 새 값을 넣는 동작이기 때문입니다.

### 이름 짓기

```javascript
var userName = "민지";
var totalPrice = 24000;
var isOpen = true;
```

- 숫자나 특수문자로 시작하지 않습니다.
- 여러 단어는 `camelCase`로 연결합니다.
- 값의 목적을 추측할 수 있는 이름을 사용합니다.
- 예약어는 변수명으로 쓰지 않습니다.

## 3. 값을 확인하는 습관

```javascript
var count = 1;
console.log(count);

count = count + 1;
console.log(count);
```

코드를 한꺼번에 작성하기보다 중요한 단계마다 값을 출력하면 오류가 생긴 지점을 좁힐 수 있습니다.

## 코드로 보기 — 주문 금액 갱신

```javascript
var itemPrice = 12000;
var itemCount = 2;
var totalPrice = itemPrice * itemCount;

console.log("초기 합계:", totalPrice);

itemCount = 3;
totalPrice = itemPrice * itemCount;

console.log("변경 합계:", totalPrice);
```

### 코드 흐름

1. 상품 가격과 수량을 저장합니다.
2. 두 값을 곱해 합계를 계산합니다.
3. 수량을 재할당합니다.
4. 합계를 다시 계산하고 출력합니다.

### 예상 결과

```text
초기 합계: 24000
변경 합계: 36000
```

## 직접 해보기

1. 도시 이름을 저장하는 `city` 변수를 선언하고 값을 출력해 보세요.
2. `level`을 1로 초기화한 뒤 2로 재할당해 보세요.
3. 가격과 개수를 각각 변수에 저장하고 총액을 출력해 보세요.

<details>
<summary>정답 보기</summary>

1. `var city = "Seoul"; console.log(city);`처럼 작성합니다.
2. `var level = 1; level = 2;`처럼 두 단계로 작성합니다.
3. `var price = 5000; var count = 3; console.log(price * count);`처럼 계산합니다.

</details>

## 헷갈리기 쉬운 포인트

| 개념 | 차이 |
|---|---|
| 선언 vs 초기화 | 공간 만들기 vs 첫 값 저장하기 |
| 재할당 vs 재선언 | 기존 값 변경 vs 같은 이름을 다시 선언 |
| 변수명 vs 문자열 | `fruit`는 이름, `"fruit"`는 문자 데이터 |
| 콘솔 출력 vs 반환 | 출력은 확인용, 반환은 호출한 곳에 결과 전달 |

## 연결되는 개념

- 다음 글: [데이터 타입과 형 변환](02-data-types-and-conversion.md)
- 전체 흐름: [OVERVIEW](OVERVIEW.md)

## 셀프 체크

- [ ] HTML과 JavaScript 파일을 연결할 수 있다.
- [ ] 선언·초기화·재할당을 구분할 수 있다.
- [ ] 목적이 드러나는 변수명을 작성할 수 있다.
- [ ] 콘솔에서 중간값을 확인할 수 있다.

### 복습 질문 및 답변

**Q1. 재할당할 때 `var`를 다시 쓰지 않는 이유는 무엇인가요?**
<details><summary>답</summary>이미 존재하는 변수의 값만 바꾸는 동작이기 때문입니다.</details>

**Q2. `console.log()`는 변수의 값을 바꾸나요?**
<details><summary>답</summary>아닙니다. 전달받은 값을 콘솔에 표시할 뿐입니다.</details>

**Q3. 변수 이름을 구체적으로 지어야 하는 이유는 무엇인가요?**
<details><summary>답</summary>값의 역할과 변경 이유를 코드만 보고 추적하기 쉬워지기 때문입니다.</details>

## 한 줄 정리

> 변수는 값을 저장하는 상자라기보다, 프로그램의 상태에 의미 있는 이름을 붙이는 도구입니다.
