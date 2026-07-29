# JavaScript 문법 용어집

이번 학습 묶음에서 사용하는 핵심 용어를 실행, 데이터, 제어 흐름, 구조화된 데이터 순서로 정리했습니다.

## 실행과 변수

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| JavaScript | 웹페이지의 상태와 동작을 표현하는 프로그래밍 언어 | [1편](01-javascript-runtime-variables.md) |
| `script` | HTML에 JavaScript를 연결하거나 직접 작성하는 태그 | [1편](01-javascript-runtime-variables.md) |
| 변수 | 값에 이름을 붙여 저장하고 다시 사용하는 공간 | [1편](01-javascript-runtime-variables.md) |
| 선언 | 변수 이름이 있는 저장 공간을 만드는 동작 | [1편](01-javascript-runtime-variables.md) |
| 초기화 | 선언한 변수에 첫 값을 저장하는 동작 | [1편](01-javascript-runtime-variables.md) |
| 재할당 | 기존 변수의 값을 새 값으로 바꾸는 동작 | [1편](01-javascript-runtime-variables.md) |

## 데이터와 변환

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| number | 계산 가능한 숫자 타입 | [2편](02-data-types-and-conversion.md) |
| string | 문자들의 순서 있는 묶음 | [2편](02-data-types-and-conversion.md) |
| boolean | `true` 또는 `false`로 표현하는 논리 타입 | [2편](02-data-types-and-conversion.md) |
| `typeof` | 값의 타입을 문자열로 알려 주는 연산자 | [2편](02-data-types-and-conversion.md) |
| 형 변환 | 값을 다른 타입으로 바꾸는 과정 | [2편](02-data-types-and-conversion.md) |
| `parseInt()` | 문자열을 정수로 해석하는 함수 | [2편](02-data-types-and-conversion.md) |

## 문자열과 배열

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| 프로퍼티 | 값이나 객체가 가진 상태 정보 | [3편](03-properties-and-methods.md) |
| 메서드 | 특정 값과 연결되어 실행되는 기능 | [3편](03-properties-and-methods.md) |
| `length` | 문자열 길이 또는 배열 요소 수 | [3편](03-properties-and-methods.md) |
| 인덱스 | 0부터 시작하는 요소 위치 번호 | [3편](03-properties-and-methods.md) |
| `push()` | 배열 끝에 요소를 추가하는 메서드 | [3편](03-properties-and-methods.md) |
| `pop()` | 배열 마지막 요소를 제거하고 반환하는 메서드 | [3편](03-properties-and-methods.md) |

## 조건과 반복

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| 산술 연산자 | 덧셈·나눗셈·나머지 같은 계산 기호 | [4편](04-operators-and-conditionals.md) |
| 비교 연산자 | 두 값을 비교해 boolean을 만드는 기호 | [4편](04-operators-and-conditionals.md) |
| 논리 연산자 | 여러 boolean 조건을 연결하거나 뒤집는 기호 | [4편](04-operators-and-conditionals.md) |
| 조건문 | 조건 결과에 따라 실행 경로를 나누는 구조 | [4편](04-operators-and-conditionals.md) |
| 반복문 | 조건이 참인 동안 코드 블록을 되풀이하는 구조 | [5편](05-loops-and-iteration.md) |
| 누적 | 반복할 때 이전 결과에 현재 값을 계속 더하거나 추가하는 패턴 | [5편](05-loops-and-iteration.md) |

## 함수와 구조화된 데이터

| 용어 | 쉬운 설명 | 관련 글 |
|---|---|---|
| 함수 | 입력을 받아 정해진 로직을 실행하는 재사용 단위 | [6편](06-functions-and-structured-data.md) |
| 매개변수 | 함수 정의에서 입력을 받을 자리 | [6편](06-functions-and-structured-data.md) |
| 인자 | 함수를 호출할 때 전달하는 실제 값 | [6편](06-functions-and-structured-data.md) |
| `return` | 함수 결과를 호출한 곳으로 돌려주는 문장 | [6편](06-functions-and-structured-data.md) |
| 배열 | 순서가 있는 여러 값을 담는 자료 구조 | [6편](06-functions-and-structured-data.md) |
| 객체 | 이름이 있는 속성으로 관련 데이터를 묶는 자료 구조 | [6편](06-functions-and-structured-data.md) |
| 경계값 | 조건이 바뀌는 지점이나 빈 입력처럼 오류가 드러나기 쉬운 값 | [7편](07-javascript-problem-solving-workshop.md) |

## 빠른 구분

| 헷갈리는 개념 | 핵심 차이 |
|---|---|
| 선언 vs 재할당 | 공간 생성 vs 기존 값 변경 |
| 문자열 vs 배열 | 문자 데이터 vs 여러 값을 담는 목록 |
| 프로퍼티 vs 메서드 | 상태 정보 vs 실행 기능 |
| 조건문 vs 반복문 | 경로 선택 vs 같은 구조 반복 |
| 출력 vs 반환 | 콘솔 확인 vs 호출한 코드에 결과 전달 |
| 배열 vs 객체 | 인덱스 기반 접근 vs 속성명 기반 접근 |

통합 적용은 [문제 해결 워크숍](07-javascript-problem-solving-workshop.md)에서 확인할 수 있습니다.
