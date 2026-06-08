# 용어집 (GLOSSARY)

이 강의에서 등장한 핵심 용어를 비전공자 눈높이로 정리했습니다. "관련 글"의 링크를 따라가면 실제 코드와 함께 더 자세히 볼 수 있습니다.

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| ---- | --------- | ------- | ------------------- |
| 반복문 | 같은 명령을 정해진 범위·조건만큼 자동으로 되풀이하는 문법 | [반복문](posts/01-loops.md) | for, while, break |
| for 문 | 리스트 같은 시퀀스에서 원소를 하나씩 꺼내 반복 | [반복문](posts/01-loops.md) | 시퀀스, range |
| range | 시작~(끝-1)까지 연속된 숫자를 만드는 도구 | [반복문](posts/01-loops.md) | for-range, list() |
| while 문 | 조건이 참인 동안 계속 반복하는 문법 | [반복문](posts/01-loops.md) | 무한루프, break |
| 무한루프 | 조건이 항상 참이라 멈추지 않고 계속 도는 상태 | [반복문](posts/01-loops.md) | while, break |
| break | 반복문을 즉시 빠져나오게 하는 명령(비상구) | [반복문](posts/01-loops.md) | if, while |
| 들여쓰기 | 반복문·조건문·함수에 속한 명령을 묶는 공백 규칙 | [반복문](posts/01-loops.md) | 코드 블록 |
| 누적 변수 | 합계·개수처럼 값을 쌓아 가기 위해 0(또는 빈 값)으로 시작하는 변수 | [반복문](posts/01-loops.md) | 합계, 평균 |
| 시퀀스 | 순서(인덱스)가 있는 자료형의 통칭(리스트·문자열·튜플 등) | [리스트와 시퀀스](posts/02-sequence-list.md) | 인덱싱, 슬라이싱 |
| 리스트 | 여러 값을 순서대로 담는 변경 가능한 그릇, 대괄호 `[]` | [리스트와 시퀀스](posts/02-sequence-list.md) | append, 튜플 |
| 인덱싱 | `자료[번호]`로 한 원소를 꺼내는 것(0부터 시작) | [리스트와 시퀀스](posts/02-sequence-list.md) | 슬라이싱, 음수 인덱스 |
| 슬라이싱 | `자료[시작:끝]`으로 구간을 잘라 내는 것(끝값 미포함) | [리스트와 시퀀스](posts/02-sequence-list.md) | 인덱싱 |
| append | 리스트 맨 뒤에 원소 하나를 추가하는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | insert, pop |
| insert | 리스트의 지정한 위치에 원소를 끼워 넣는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | append |
| remove | 리스트에서 처음 나오는 특정 값을 지우는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | pop |
| pop | 원소를 제거하면서 그 값을 돌려주는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | remove |
| sort | 리스트를 오름차순(문자열은 사전순)으로 정렬하는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | sorted |
| count | 시퀀스 안에 특정 값이 몇 개 있는지 세는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | 단어 빈도 |
| split | 문자열을 기준 문자로 쪼개 리스트로 만드는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | join |
| join | 리스트를 기준 문자로 이어 붙여 문자열로 만드는 메서드 | [리스트와 시퀀스](posts/02-sequence-list.md) | split |
| 튜플 | 한 번 만들면 못 바꾸는(불변) 묶음 자료형, 소괄호 `()` | [튜플](posts/03-tuple.md) | 불변성, 리스트 |
| 불변성(immutable) | 만든 뒤 내용을 바꿀 수 없는 성질 | [튜플](posts/03-tuple.md) | 튜플, 딕셔너리 Key |
| 딕셔너리 | Key–Value 짝으로 자료를 보관하는 자료형, 중괄호 `{}` | [딕셔너리](posts/04-dictionary.md) | Key, Value, items |
| Key | 딕셔너리에서 값을 꺼내는 이름표(열쇠) | [딕셔너리](posts/04-dictionary.md) | Value, 불변성 |
| Value | Key에 연결되어 저장된 실제 값 | [딕셔너리](posts/04-dictionary.md) | Key |
| keys/values/items | 딕셔너리의 Key·Value·(Key,Value)쌍을 꺼내는 메서드 | [딕셔너리](posts/04-dictionary.md) | for, 뷰 객체 |
| zip | 여러 묶음의 같은 위치 원소를 짝지어 묶는 명령어 | [딕셔너리](posts/04-dictionary.md) | dict(), 언패킹 |
| 단어 빈도 | 각 단어가 몇 번 나오는지 딕셔너리로 집계하는 작업 | [딕셔너리](posts/04-dictionary.md) | split, 조건문 |
| 함수 | 자주 쓰는 동작을 이름 붙여 재사용하는 코드 단위 | [함수와 메서드](posts/05-functions.md) | return, 매개변수 |
| 내장 함수 | 파이썬이 기본 제공하는 함수(len, sum, max 등) | [함수와 메서드](posts/05-functions.md) | 메서드 |
| 사용자 정의 함수 | `def`로 직접 만든 함수 | [함수와 메서드](posts/05-functions.md) | return, 매개변수 |
| return | 함수의 결과를 함수 밖으로 넘겨주는 키워드 | [함수와 메서드](posts/05-functions.md) | None, print |
| 매개변수 | 함수가 입력으로 받는 값(들) | [함수와 메서드](posts/05-functions.md) | 기본값, 에러 메시지 |
| 메서드 | 자료 뒤에 점을 찍어 부르는 동작(`자료.동작()`) | [함수와 메서드](posts/05-functions.md) | 함수 |
| 지역 변수 | 함수 안에서만 살아 있는 변수 | [함수와 메서드](posts/05-functions.md) | 전역 변수, 변수 범위 |
| 전역 변수 | 함수 밖에서 만들어 함수 안에서도 읽을 수 있는 변수 | [함수와 메서드](posts/05-functions.md) | 지역 변수 |
| 타입 힌트 | 매개변수·반환값의 자료형을 표기하는 안내(`a: int -> int`) | [함수와 메서드](posts/05-functions.md) | docstring |
| docstring | 함수 바로 아래 삼중따옴표로 적는 함수 설명 문서 | [함수와 메서드](posts/05-functions.md) | help(), 타입 힌트 |
| None | "값이 없음"을 뜻하는 특수한 값(return 없는 함수의 결과) | [함수와 메서드](posts/05-functions.md) | return |
