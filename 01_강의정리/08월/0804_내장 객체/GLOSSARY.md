# JavaScript 내장 객체 용어집

내장 객체와 계산 실습에서 자주 만나는 용어를 쉬운 설명과 함께 정리했습니다.

## 브라우저 환경

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 내장 객체 | 언어 또는 실행 환경이 미리 제공하는 데이터와 기능의 묶음 | [브라우저 내장 객체](01-browser-built-in-objects.md) | 메서드, 프로퍼티 |
| `window` | 브라우저 탭의 전역 환경과 창 기능을 나타내는 객체 | [브라우저 내장 객체](01-browser-built-in-objects.md) | 전역 객체 |
| `document` | 현재 HTML 문서를 객체 트리로 나타내는 `window`의 프로퍼티 | [브라우저 내장 객체](01-browser-built-in-objects.md) | DOM |
| `innerWidth` | 브라우저 콘텐츠 영역의 현재 너비 | [브라우저 내장 객체](01-browser-built-in-objects.md) | 반응형 UI |
| `window.open()` | URL을 새 탐색 컨텍스트로 여는 메서드 | [브라우저 내장 객체](01-browser-built-in-objects.md) | 팝업 차단 |

## 숫자와 수학

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `Number()` | 값을 숫자 타입으로 변환하는 함수 | [Number와 Math](02-number-and-math.md) | 형 변환 |
| `NaN` | 숫자 계산 결과를 만들 수 없음을 나타내는 특별한 숫자 값 | [Number와 Math](02-number-and-math.md) | `Number.isNaN()` |
| `toFixed()` | 지정한 소수 자릿수로 반올림한 문자열을 반환하는 메서드 | [Number와 Math](02-number-and-math.md) | 표시 포맷 |
| `toLocaleString()` | 지역 표기 규칙에 맞는 숫자 문자열을 만드는 메서드 | [Number와 Math](02-number-and-math.md) | 천 단위 구분 |
| `Math.floor()` | 주어진 값보다 크지 않은 가장 큰 정수를 반환하는 내림 함수 | [Number와 Math](02-number-and-math.md) | `round`, `ceil` |
| `Math.hypot()` | 여러 값의 제곱합에 대한 제곱근을 구하는 메서드 | [Number와 Math](02-number-and-math.md) | 거리 공식 |
| 의사난수 | `Math.random()`이 생성하는 0 이상 1 미만의 예측 불가능해 보이는 값 | [Number와 Math](02-number-and-math.md) | 범위 변환 |

## 날짜와 시간

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `Date` | 특정 시점을 나타내고 날짜 구성요소를 조회·변경하는 내장 객체 | [Date와 시간 계산](03-date-and-time.md) | 타임스탬프 |
| 타임스탬프 | 기준 시점부터 흐른 시간을 밀리초 숫자로 표현한 값 | [Date와 시간 계산](03-date-and-time.md) | `getTime()` |
| 월 인덱스 | `getMonth()`가 1월을 0, 12월을 11로 반환하는 규칙 | [Date와 시간 계산](03-date-and-time.md) | `+ 1` |
| 상대시간 | 절대 날짜 대신 ‘5분 전’처럼 현재와의 차이로 표현한 시간 | [Date와 시간 계산](03-date-and-time.md) | 단위 환산 |
| `setInterval()` | 일정 시간 간격으로 함수를 반복 호출하는 타이머 | [Date와 시간 계산](03-date-and-time.md) | `clearInterval()` |

## 문자열과 데이터 교환

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 문자열 불변성 | 문자열 메서드가 원본을 직접 바꾸지 않고 새 문자열을 반환하는 성질 | [String과 JSON](04-string-and-json.md) | 재할당 |
| `trim()` | 문자열 양끝의 공백을 제거한 새 문자열을 반환하는 메서드 | [String과 JSON](04-string-and-json.md) | 입력 정리 |
| `split()` | 구분자를 기준으로 문자열을 배열로 나누는 메서드 | [String과 JSON](04-string-and-json.md) | `join()` |
| 직렬화 | JavaScript 값을 저장·전송 가능한 JSON 문자열로 변환하는 과정 | [String과 JSON](04-string-and-json.md) | `JSON.stringify()` |
| 역직렬화 | JSON 문자열을 JavaScript 값으로 복원하는 과정 | [String과 JSON](04-string-and-json.md) | `JSON.parse()` |
| JSON | 문자열 키와 제한된 값 타입을 사용하는 데이터 교환 형식 | [String과 JSON](04-string-and-json.md) | API |

## 계산 설계

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 순수 함수 | 같은 입력이면 같은 결과를 반환하고 외부 화면을 직접 바꾸지 않는 함수 | [계산기 설계 패턴](05-calculator-design-patterns.md) | 테스트 |
| 복리 | 이자에 다시 이자가 붙는 방식 | [계산기 설계 패턴](05-calculator-design-patterns.md) | 거듭제곱 |
| 수익률 | 투입 금액 대비 손익이 차지하는 비율 | [계산기 설계 패턴](05-calculator-design-patterns.md) | 백분율 |
| 손익분기점 | 수익과 총비용이 같아 이익이 0이 되는 지점 | [계산기 설계 패턴](05-calculator-design-patterns.md) | 공헌이익률 |

