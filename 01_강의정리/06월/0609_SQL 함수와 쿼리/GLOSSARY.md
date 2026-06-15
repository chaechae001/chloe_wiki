# 용어집 (GLOSSARY)

이 강의에서 나온 핵심 용어를 비전공자 눈높이로 정리했다. 헷갈리는 용어가 나오면 여기서 빠르게 확인하자.

| 용어 | 쉬운 설명 | 관련 강의명 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| 집계 함수 (Aggregate Function) | 여러 행을 하나의 대표값으로 압축하는 함수의 통칭 | 집계 함수와 LIMIT | COUNT, SUM, AVG, GROUP BY |
| COUNT | 데이터(행)의 개수를 세는 함수. 컬럼을 지정하면 NULL은 제외 | 집계 함수와 LIMIT | COUNT(*), NULL, DISTINCT |
| SUM | 지정한 숫자 컬럼의 값을 모두 더해 총합을 구하는 함수 | 집계 함수와 LIMIT | AVG, GROUP BY |
| AVG | 지정한 컬럼의 평균을 구하는 함수. NULL은 계산에서 제외 | 집계 함수와 LIMIT | SUM, COUNT |
| MAX / MIN | 최댓값/최솟값을 가져오는 함수. 숫자뿐 아니라 문자(사전 순)도 가능 | 집계 함수와 LIMIT | 서브쿼리, ORDER BY |
| NULL | "값이 없음"을 뜻하는 특수 상태. 0이나 빈 문자열과 다름 | 집계 함수와 LIMIT / LEFT JOIN과 NULL | IS NULL, COUNT |
| LIMIT | 출력 행의 개수를 제한하는 명령. 시작 위치는 0부터 셈 | 집계 함수와 LIMIT | 페이지네이션, ORDER BY |
| GROUP BY | 같은 값을 가진 행끼리 묶어 그룹마다 집계하는 절 | GROUP BY와 HAVING | 집계 함수, HAVING |
| HAVING | GROUP BY로 묶은 그룹의 집계 결과에 조건을 거는 절 | GROUP BY와 HAVING | WHERE, GROUP BY |
| WHERE | 그룹으로 묶기 전, 개별 행에 조건을 거는 절 | GROUP BY와 HAVING | HAVING, IS NULL |
| ORDER BY | 결과를 특정 컬럼 기준으로 정렬하는 절 (ASC 오름/DESC 내림) | GROUP BY와 HAVING | LIMIT, GROUP BY |
| JOIN | 나뉜 여러 테이블을 공통 키로 이어 붙여 함께 조회하는 명령 | INNER JOIN | 외래키, ON, 정규화 |
| INNER JOIN | 양쪽 테이블에 매칭되는 키가 있는 행만(교집합) 가져오는 조인 | INNER JOIN | LEFT JOIN, ON |
| ON | 조인할 두 테이블을 어떤 키로 연결할지 지정하는 조건 절 | INNER JOIN | INNER JOIN, 외래키 |
| 외래키 (Foreign Key) | 다른 테이블의 키를 가리켜 두 테이블을 연결하는 컬럼 | INNER JOIN | 기본키, 정규화, JOIN |
| 기본키 (Primary Key) | 한 테이블에서 각 행을 고유하게 식별하는 컬럼 | INNER JOIN | 외래키, JOIN |
| 정규화 | 중복·수정 오류·무결성 문제를 줄이려 테이블을 나누는 설계 원칙 | INNER JOIN | JOIN, 외래키 |
| ambiguous column name | 조인 시 양쪽에 같은 이름 컬럼이 있어 어느 것인지 모호할 때 나는 오류 | INNER JOIN | 테이블명.컬럼명, 별칭 |
| 테이블명.컬럼명 | 어느 테이블의 컬럼인지 명시하는 표기. 모호성 오류를 막음 | INNER JOIN | 별칭, JOIN |
| 별칭 (alias) | 테이블·컬럼에 짧은 이름을 붙이는 것 (AS). 가독성을 높임 | INNER JOIN / 서브쿼리 | 테이블명.컬럼명 |
| LEFT JOIN | 왼쪽 테이블의 모든 행을 남기고, 짝이 없으면 NULL로 채우는 조인 | LEFT JOIN과 NULL | INNER JOIN, IS NULL |
| RIGHT JOIN | 오른쪽 테이블의 모든 행을 기준으로 하는 조인. 보통 LEFT로 대체 | LEFT JOIN과 NULL | LEFT JOIN |
| IS NULL / IS NOT NULL | 값이 비어 있는/채워진 행을 골라내는 조건. `= NULL`은 동작 안 함 | LEFT JOIN과 NULL | NULL, LEFT JOIN |
| 데이터 정의서 (Data Dictionary) | 각 컬럼이 무엇을 뜻하는지 정리한 문서. 조인 키 파악에 도움 | LEFT JOIN과 NULL | ERD, 외래키 |
| ERD | 테이블 간 관계(키 연결)를 그림으로 나타낸 관계도 | LEFT JOIN과 NULL | 외래키, JOIN |
| 서브쿼리 (Subquery) | 하나의 쿼리 안에 포함된 또 다른 쿼리. 메인 쿼리보다 먼저 실행됨 | 서브쿼리 | 단일/다중 행, 스칼라 서브쿼리 |
| 메인 쿼리 | 서브쿼리를 품고 있는 바깥쪽 쿼리 | 서브쿼리 | 서브쿼리 |
| 단일 행 서브쿼리 | 결과가 한 값으로 나오는 서브쿼리. 비교 연산자(=, >, …) 사용 | 서브쿼리 | 다중 행 서브쿼리 |
| 다중 행 서브쿼리 | 결과가 여러 값으로 나오는 서브쿼리. IN·ANY·ALL 사용 | 서브쿼리 | IN, ANY, ALL |
| IN | 목록 중 하나라도 같으면 만족하는 다중 행 연산자 | 서브쿼리 | ANY, ALL |
| ANY | 목록 중 하나라도 비교가 성립하면 참인 연산자 | 서브쿼리 | ALL, IN |
| ALL | 목록 전부에 대해 비교가 성립해야 참인 연산자 | 서브쿼리 | ANY, IN |
| 스칼라 서브쿼리 | SELECT 절에서 쓰며 한 행당 한 값을 돌려주는 서브쿼리 | 서브쿼리 | JOIN, 서브쿼리 |
| DISTINCT | 중복을 제거하고 고유한 값만 남기는 키워드 | 집계 함수와 LIMIT | COUNT |
