# 용어집

실행 컨텍스트와 `this`를 이해할 때 자주 만나는 용어를 쉬운 말로 정리했습니다.

## 실행 환경과 스코프

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 실행 컨텍스트 | 현재 코드를 실행하는 데 필요한 변수, 함수, 상위 환경, `this` 등의 정보를 묶어 관리하는 실행 단위 | [실행 컨텍스트와 렉시컬 환경](01-execution-context-and-lexical-environment.md) | 코드 평가 |
| 렉시컬 환경 | 현재 스코프의 식별자 정보와 바깥 스코프로 이어지는 참조를 담은 구조 | [실행 컨텍스트와 렉시컬 환경](01-execution-context-and-lexical-environment.md) | 스코프 체인 |
| 환경 레코드 | 현재 스코프에 선언된 변수와 함수의 이름·상태·값을 기록하는 부분 | [실행 컨텍스트와 렉시컬 환경](01-execution-context-and-lexical-environment.md) | 식별자 |
| 외부 렉시컬 환경 참조 | 현재 환경에 이름이 없을 때 찾아갈 상위 환경의 연결 정보 | [콜 스택과 스코프 체인](02-call-stack-scope-chain-and-closures.md) | 렉시컬 스코프 |
| 콜 스택 | 실행 중인 컨텍스트를 후입선출 방식으로 쌓고 제거하는 구조 | [콜 스택과 스코프 체인](02-call-stack-scope-chain-and-closures.md) | LIFO |
| 스코프 체인 | 현재 환경부터 바깥 환경으로 식별자를 차례로 찾는 경로 | [콜 스택과 스코프 체인](02-call-stack-scope-chain-and-closures.md) | 클로저 |
| 클로저 | 함수가 자신이 정의된 바깥 렉시컬 환경의 변수를 계속 사용할 수 있는 성질 | [콜 스택과 스코프 체인](02-call-stack-scope-chain-and-closures.md) | 상태 보존 |

## `this`와 함수 호출

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `this` | 일반 함수에서는 호출 방식에 따라 현재 호출의 기준 객체를 가리키는 값 | [`this`와 호출 방식](03-this-binding-and-method-calls.md) | 바인딩 |
| 메서드 호출 | `object.method()`처럼 객체를 통해 함수를 호출하여 `this`가 그 객체를 가리키게 하는 방식 | [`this`와 호출 방식](03-this-binding-and-method-calls.md) | 객체 상태 |
| 명시적 바인딩 | `call`, `apply`, `bind`로 함수가 사용할 `this`를 직접 지정하는 방식 | [`call`·`apply`·`bind`](04-call-apply-and-bind.md) | 함수 재사용 |
| `call()` | `this`와 인자를 쉼표로 전달해 함수를 즉시 실행하는 메서드 | [`call`·`apply`·`bind`](04-call-apply-and-bind.md) | 가변 인자 |
| `apply()` | `this`와 인자를 배열 형태로 전달해 함수를 즉시 실행하는 메서드 | [`call`·`apply`·`bind`](04-call-apply-and-bind.md) | 배열 |
| `bind()` | 지정한 `this`가 고정된 새 함수를 만들고 실행은 나중으로 미루는 메서드 | [`call`·`apply`·`bind`](04-call-apply-and-bind.md) | 부분 적용 |
| 콜백 함수 | 다른 함수에 전달되어 그 함수가 정한 시점과 방식으로 호출하는 함수 | [콜백과 렉시컬 `this`](05-callbacks-arrow-functions-and-events.md) | 이벤트 핸들러 |
| 렉시컬 `this` | 화살표 함수가 자체 `this`를 만들지 않고 바깥 스코프의 값을 사용하는 규칙 | [콜백과 렉시컬 `this`](05-callbacks-arrow-functions-and-events.md) | 화살표 함수 |
| `currentTarget` | 이벤트 리스너가 등록된 요소를 명시적으로 가리키는 이벤트 객체의 프로퍼티 | [콜백과 렉시컬 `this`](05-callbacks-arrow-functions-and-events.md) | DOM 이벤트 |

## 선언과 초기화

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 호이스팅 | 코드 실행 전 평가 단계에서 선언 정보가 환경에 먼저 등록되어 나타나는 현상 | [호이스팅과 TDZ](06-hoisting-and-tdz.md) | 코드 평가 |
| 초기화 | 선언된 식별자가 실제로 참조 가능한 첫 값을 얻는 과정 | [호이스팅과 TDZ](06-hoisting-and-tdz.md) | `undefined` |
| TDZ | `let`·`const`가 환경에 등록된 뒤 초기화되기 전까지 접근할 수 없는 구간 | [호이스팅과 TDZ](06-hoisting-and-tdz.md) | `ReferenceError` |
| 함수 선언문 | 평가 단계에서 함수 객체까지 준비되어 선언보다 앞에서도 호출할 수 있는 함수 정의 방식 | [호이스팅과 TDZ](06-hoisting-and-tdz.md) | 함수 표현식 |
