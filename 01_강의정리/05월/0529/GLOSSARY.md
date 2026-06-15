# 용어집 (Glossary)

이 강의에서 등장한 핵심 용어를 한눈에 정리했습니다.

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| print() | 괄호 안의 값을 화면에 보여주는 출력 명령 | [출력 — print()](posts/01-print-output.md) | input(), f-string |
| input() | 사용자에게 값을 입력받는 명령. 결과는 항상 문자열 | [입력과 형 변환](posts/05-input-typecasting.md) | 형 변환, TypeError |
| 자료형 | 값의 종류(숫자·글자·묶음 등) | [기본 자료형](posts/02-basic-datatypes.md) | type() |
| 정수 (int) | 소수점 없는 숫자 | [기본 자료형](posts/02-basic-datatypes.md) | float, 형 변환 |
| 실수 (float) | 소수점 있는 숫자. `/` 나눗셈 결과도 실수 | [기본 자료형](posts/02-basic-datatypes.md) | int, 사칙연산 |
| 문자열 (str) | 따옴표로 감싼 글자 데이터 | [기본 자료형](posts/02-basic-datatypes.md) | 시퀀스, 문자열 연산 |
| 리스트 (list) | `[]` 안에 여러 값을 순서대로 담는 묶음 | [기본 자료형](posts/02-basic-datatypes.md) | 리스트 메서드, 인덱싱 |
| 논리형 (bool) | 참(True)/거짓(False) 두 값만 갖는 자료형 | [논리형과 비교·논리 연산](posts/09-boolean-comparison.md) | 비교 연산자, 조건문 |
| type() | 값의 자료형을 확인하는 함수 | [기본 자료형](posts/02-basic-datatypes.md) | 자료형 |
| 주석 (Comment) | 컴퓨터가 무시하는 설명용 메모(`#`, `"""..."""`) | [기본 자료형](posts/02-basic-datatypes.md) | 가독성 |
| 변수 (Variable) | 값을 담아 이름으로 꺼내 쓰는 그릇 | [변수](posts/03-variables.md) | 대입, 덮어쓰기 |
| 대입 (=) | 오른쪽 값을 왼쪽 변수에 담는 것 | [변수](posts/03-variables.md) | 비교 연산자 ==, 덮어쓰기 |
| 스네이크 표기법 | 단어를 `_`로 잇는 이름 규칙(파이썬 권장) | [변수](posts/03-variables.md) | 카멜 표기법, 네이밍 |
| 사칙연산 | 더하기·빼기·곱하기·나누기(`+ - * /`) | [자료형의 연산](posts/04-operators.md) | 특수연산 |
| 몫 (//) | 나눗셈의 정수 몫만 구하는 연산 | [자료형의 연산](posts/04-operators.md) | 나머지, 나누기 |
| 나머지 (%) | 나누고 남은 값을 구하는 연산(홀짝 판별 등) | [자료형의 연산](posts/04-operators.md) | 몫, 조건문 |
| 제곱 (**) | 거듭제곱을 구하는 연산 | [자료형의 연산](posts/04-operators.md) | 사칙연산 |
| 문자열 연산 | `+`(이어붙이기), `*`(반복) | [자료형의 연산](posts/04-operators.md) | 시퀀스, 형 변환 |
| 형 변환 (Type Casting) | 값의 자료형을 바꾸는 것(`int` `float` `str` `list`) | [입력과 형 변환](posts/05-input-typecasting.md) | input(), TypeError |
| TypeError | 자료형이 안 맞아 연산할 수 없을 때 나는 오류 | [입력과 형 변환](posts/05-input-typecasting.md) | 형 변환 |
| 인덱스 (index) | 원소의 위치 번호. 0부터 시작 | [인덱싱과 슬라이싱](posts/06-indexing-slicing.md) | 인덱싱, 슬라이싱 |
| 인덱싱 (Indexing) | `자료[i]`로 특정 위치 원소 하나 꺼내기 | [인덱싱과 슬라이싱](posts/06-indexing-slicing.md) | 음수 인덱스, 슬라이싱 |
| 음수 인덱스 | 뒤에서부터 세는 인덱스(`-1`이 마지막) | [인덱싱과 슬라이싱](posts/06-indexing-slicing.md) | 인덱싱 |
| 슬라이싱 (Slicing) | `자료[시작:끝]`으로 범위를 잘라 가져오기(끝은 미만) | [인덱싱과 슬라이싱](posts/06-indexing-slicing.md) | 인덱싱 |
| 중첩 리스트 | 리스트 안의 리스트. `[i][j]`로 접근 | [인덱싱과 슬라이싱](posts/06-indexing-slicing.md) | 인덱싱 |
| append() | 리스트 끝에 원소 하나 추가 | [리스트 메서드](posts/07-list-methods.md) | insert, extend |
| insert() | 지정한 위치에 원소 삽입 | [리스트 메서드](posts/07-list-methods.md) | append |
| remove() | 처음 등장하는 값 하나 삭제 | [리스트 메서드](posts/07-list-methods.md) | 인덱스, del |
| sort() | 리스트를 정렬(오름차순, `reverse=True`면 내림차순) | [리스트 메서드](posts/07-list-methods.md) | sorted |
| 시퀀스 자료형 | 순서가 있는 자료형(문자열·리스트) | [시퀀스 자료형](posts/08-sequence-types.md) | 인덱싱, in, len |
| in 연산자 | 원소 포함 여부를 참/거짓으로 확인 | [시퀀스 자료형](posts/08-sequence-types.md) | 논리형, 조건문 |
| len() | 시퀀스의 길이(원소 개수)를 구하는 함수 | [시퀀스 자료형](posts/08-sequence-types.md) | 시퀀스 |
| AttributeError | 해당 자료형에 없는 기능(메서드)을 부를 때 나는 오류 | [시퀀스 자료형](posts/08-sequence-types.md) | append, 공식 문서 |
| 비교 연산자 | 두 값을 견주어 참/거짓 반환(`== != < > <= >=`) | [논리형과 비교·논리 연산](posts/09-boolean-comparison.md) | 논리형, 조건문 |
| and | 모든 조건이 참이어야 참 | [논리형과 비교·논리 연산](posts/09-boolean-comparison.md) | or, not |
| or | 하나라도 참이면 참 | [논리형과 비교·논리 연산](posts/09-boolean-comparison.md) | and, not |
| not | 논리값을 뒤집음(참↔거짓) | [논리형과 비교·논리 연산](posts/09-boolean-comparison.md) | and, or |
| 조건문 | 조건에 따라 실행 명령이 달라지는 구문 | [조건문](posts/10-conditionals.md) | if, elif, else |
| if / elif / else | 참일 때 / 다른 조건일 때 / 그 외 실행 | [조건문](posts/10-conditionals.md) | 비교 연산자, 들여쓰기 |
| 들여쓰기 | 블록의 범위를 칸으로 구분하는 파이썬 핵심 문법 | [조건문](posts/10-conditionals.md) | if, 콜론 |
