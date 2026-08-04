# ES6와 배열 메서드 용어집

현대 JavaScript 문법과 배열 처리에서 자주 만나는 용어를 역할 중심으로 정리했습니다.

## 변수와 스코프

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `let` | 같은 블록 안에서 다시 선언할 수 없고 값은 재할당할 수 있는 변수 선언 | [`let`·`const`와 스코프](01-let-const-and-scope.md) | 블록 스코프 |
| `const` | 선언과 동시에 초기화하며 다른 값으로 재할당할 수 없는 변수 선언 | [`let`·`const`와 스코프](01-let-const-and-scope.md) | 객체 변경 |
| 스코프 | 식별자를 참조할 수 있는 코드의 유효 범위 | [`let`·`const`와 스코프](01-let-const-and-scope.md) | 전역·지역 스코프 |
| 스코프 체인 | 현재 범위에서 찾지 못한 식별자를 바깥 범위로 차례로 탐색하는 연결 구조 | [`let`·`const`와 스코프](01-let-const-and-scope.md) | 렉시컬 스코프 |
| 렉시컬 스코프 | 함수가 호출된 곳이 아니라 정의된 위치를 기준으로 상위 스코프가 정해지는 규칙 | [`let`·`const`와 스코프](01-let-const-and-scope.md) | 클로저 |

## ES6 표현 문법

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 화살표 함수 | `=>`를 사용해 함수를 짧게 표현하는 문법 | [화살표 함수와 간결한 표현](02-arrow-functions-and-modern-expressions.md) | 콜백 |
| 템플릿 리터럴 | 백틱과 `${}`로 값과 여러 줄 문자열을 자연스럽게 조합하는 문법 | [화살표 함수와 간결한 표현](02-arrow-functions-and-modern-expressions.md) | 문자열 보간 |
| 옵셔널 체이닝 | 접근 대상이 없을 때 오류 대신 `undefined`를 반환하는 `?.` 문법 | [화살표 함수와 간결한 표현](02-arrow-functions-and-modern-expressions.md) | nullish 값 |
| 구조 분해 할당 | 객체의 이름이나 배열의 위치를 기준으로 값을 변수에 꺼내는 문법 | [구조 분해와 Rest·Spread](03-destructuring-rest-and-spread.md) | 기본값 |
| Rest | 여러 값을 하나의 배열이나 객체에 모으는 `...` 문법 | [구조 분해와 Rest·Spread](03-destructuring-rest-and-spread.md) | 나머지 매개변수 |
| Spread | 배열·문자열·객체의 값을 개별 항목으로 펼치는 `...` 문법 | [구조 분해와 Rest·Spread](03-destructuring-rest-and-spread.md) | 얕은 복사 |

## 함수와 배열 처리

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 고차 함수 | 함수를 인자로 받거나 함수를 반환하는 함수 | [고차 함수와 콜백](04-higher-order-functions-and-callbacks.md) | 함수 합성 |
| 콜백 함수 | 다른 함수에 전달되어 정해진 시점에 호출되는 함수 | [고차 함수와 콜백](04-higher-order-functions-and-callbacks.md) | 이벤트 핸들러 |
| `forEach()` | 각 요소마다 작업을 실행하지만 의미 있는 반환값은 만들지 않는 메서드 | [`forEach`·`map`·`filter`](05-foreach-map-and-filter.md) | 부수 효과 |
| `map()` | 각 요소의 변환 결과를 모아 같은 길이의 새 배열을 만드는 메서드 | [`forEach`·`map`·`filter`](05-foreach-map-and-filter.md) | 변환 |
| `filter()` | 조건이 참인 요소만 모아 새 배열을 만드는 메서드 | [`forEach`·`map`·`filter`](05-foreach-map-and-filter.md) | 불리언 조건 |
| `reduce()` | 누적값과 현재 요소를 결합해 하나의 결과로 줄이는 메서드 | [`reduce`와 배열 메서드의 원리](06-reduce-and-array-method-internals.md) | 초기값 |
| 누적값 | `reduce()`가 이전 단계의 결과를 다음 단계로 전달하는 값 | [`reduce`와 배열 메서드의 원리](06-reduce-and-array-method-internals.md) | accumulator |
