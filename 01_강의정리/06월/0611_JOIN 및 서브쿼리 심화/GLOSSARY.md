# 용어집 (GLOSSARY)

JOIN 심화 · 서브쿼리 심화 · Python 연동에서 등장한 핵심 용어를 비전공자 눈높이로 정리했습니다.

| 용어 | 쉬운 설명 | 관련 강의명 | 함께 보면 좋은 개념 |
| ---- | --------- | ----------- | ----------- |
| JOIN | 여러 표를 조건으로 이어 붙여 하나의 넓은 결과로 만드는 것 | ① JOIN 기초 | 기본키-외래키, INNER JOIN |
| EQUI JOIN | 등호(=)로 "정확히 같은 값"끼리 잇는 조인 | ① JOIN 기초 | Non-EQUI JOIN |
| Non-EQUI JOIN | `>`, `<`, `BETWEEN` 등 비교 연산자로 범위·대소로 잇는 조인 | ① JOIN 기초 | EQUI JOIN, CASE |
| INNER JOIN | 양쪽 표에 짝이 있는 행만 남기는 기본 조인(교집합) | ① JOIN 기초 | ON절, OUTER JOIN |
| ON절 | 조인 조건을 직접 지정하는 구문 — 컬럼명이 달라도 조인 가능 | ① JOIN 기초 | WHERE, USING |
| 기본키(PK) | 한 표에서 행을 유일하게 구분하는 컬럼 | ① JOIN 기초 | 외래키, EQUI JOIN |
| 외래키(FK) | 다른 표의 기본키를 가리키는 연결용 컬럼 | ① JOIN 기초 | 기본키, JOIN |
| USING절 | 두 표에 이름이 같은 컬럼이 있을 때 그 컬럼으로 간단히 잇는 축약 문법(별칭 금지) | ② USING·NATURAL·CROSS | ON절, NATURAL JOIN |
| NATURAL JOIN | 이름이 같은 모든 컬럼을 자동으로 조인 조건으로 삼는 조인(별칭 금지, 위험) | ② USING·NATURAL·CROSS | USING절 |
| CROSS JOIN | 조건 없이 모든 행 조합을 만드는 조인(결과 = A×B) | ② USING·NATURAL·CROSS | 카티션 곱 |
| 카티션 곱 | 두 집합의 가능한 모든 짝을 만든 결과(곱집합) | ② USING·NATURAL·CROSS | CROSS JOIN |
| OUTER JOIN | 교집합에 더해 한쪽에만 있는 행까지 NULL로 채워 포함하는 조인 | ③ OUTER JOIN | LEFT/RIGHT/FULL, NULL |
| LEFT JOIN | 왼쪽(기준) 표를 전부 남기고, 오른쪽 짝이 없으면 NULL | ③ OUTER JOIN | 기준 테이블, IS NULL |
| RIGHT JOIN | 오른쪽 표를 전부 남기는 조인(순서 바꾸면 LEFT와 동일) | ③ OUTER JOIN | LEFT JOIN |
| FULL OUTER JOIN | 양쪽 표를 모두 남기는 조인 | ③ OUTER JOIN | UNION |
| NULL | "값이 없음" 자체 — 0이나 빈 문자열과 다르며 `IS NULL`로만 비교 | ③ OUTER JOIN | IS NULL, COALESCE |
| IS NULL | 값이 NULL인지 확인하는 연산(없는 데이터 찾기의 핵심) | ③ OUTER JOIN | LEFT JOIN, NOT EXISTS |
| 기준 테이블 | OUTER JOIN에서 "전부 다 보고 싶은" 쪽 표(보통 왼쪽에 둠) | ③ OUTER JOIN | LEFT JOIN |
| 셀프 조인 | 한 표를 별칭 두 개로 불러 자기 자신과 잇는 조인(별칭 필수) | ④ 셀프 조인 | 별칭, 계층형 질의 |
| 별칭(alias) | 표·컬럼에 임시 이름을 붙여 구분하는 것 | ④ 셀프 조인 | 셀프 조인 |
| 계층형 질의 | 상사-부하, 부모-자식처럼 단계 구조를 펼쳐 보는 질의 | ④ 셀프 조인 | 셀프 조인, 자기참조 FK |
| 자기참조 외래키 | 같은 표의 다른 행을 가리키는 외래키(예: 관리자 번호) | ④ 셀프 조인 | 외래키, 계층형 질의 |
| 서브쿼리 | 쿼리 안에 들어가는 또 다른 쿼리(쿼리 속의 쿼리) | ⑤ 동작 방식 | 메인쿼리, 스칼라 |
| 메인쿼리 | 서브쿼리를 감싸는 바깥쪽 쿼리 | ⑤ 동작 방식 | 서브쿼리 |
| 비연관 서브쿼리 | 메인쿼리 컬럼을 참조하지 않아 한 번만 실행되는 서브쿼리 | ⑤ 동작 방식 | 연관 서브쿼리 |
| 연관 서브쿼리 | 메인쿼리 컬럼을 참조해 행마다 다시 실행되는 서브쿼리 | ⑤ 동작 방식 | EXISTS, 비연관 |
| EXISTS | 서브쿼리 결과가 하나라도 있으면 참으로 보는 연산자 | ⑤ 동작 방식 | IN, NOT EXISTS |
| 단일 행 서브쿼리 | 값 하나(1행 1컬럼)를 반환 → =, >, < 등과 사용 | ⑥ 반환 형태 | 스칼라 서브쿼리 |
| 다중 행 서브쿼리 | 여러 행을 반환 → IN, EXISTS, ALL, ANY와 사용 | ⑥ 반환 형태 | IN, ALL, ANY |
| 다중 컬럼 서브쿼리 | 여러 컬럼을 묶어 비교(`(col1,col2) IN ...`)로 그룹별 대표 행 추출 | ⑥ 반환 형태 | GROUP BY, 단일 행 |
| IN | 여러 후보값 중 하나와 일치하면 참 | ⑥ 반환 형태 | NOT IN, EXISTS |
| NOT IN | 후보값 목록에 없으면 참(NULL 섞이면 주의) | ⑥ 반환 형태 | NOT EXISTS |
| ALL | 서브쿼리의 모든 값에 대해 조건을 만족해야 참(SQLite는 MAX로 대체) | ⑥ 반환 형태 | ANY, MAX |
| ANY | 서브쿼리 값 중 하나 이상이 조건을 만족하면 참(SQLite는 MIN으로 대체) | ⑥ 반환 형태 | ALL, MIN |
| 스칼라 서브쿼리 | 값 하나만 반환해 SELECT·WHERE·HAVING의 값 자리에 끼우는 서브쿼리 | ⑦ 스칼라 서브쿼리 | 단일 행, COALESCE |
| HAVING | 그룹(GROUP BY) 집계 결과에 거는 조건절 | ⑦ 스칼라 서브쿼리 | GROUP BY, WHERE |
| COALESCE | NULL을 지정한 기본값으로 바꿔 주는 함수 | ⑦ 스칼라 서브쿼리 | NULL, IS NULL |
| CAST | 값의 자료형을 바꾸는 것(예: 정수→실수)으로 나눗셈 오류 방지 | ⑦ 스칼라 서브쿼리 | 비율 계산 |
| DUAL | 행 하나뿐인 더미 테이블(SQLite엔 없어 FROM 생략) | ⑦ 스칼라 서브쿼리 | 스칼라 서브쿼리 |
| VIEW(뷰) | 데이터를 저장하지 않고 쿼리에 이름을 붙여 표처럼 쓰는 가상 테이블 | ⑧ 뷰 | 편리성·독립성·보안성 |
| 가상 테이블 | 실제 저장 없이 논리적으로만 존재하는 테이블(뷰) | ⑧ 뷰 | VIEW |
| CREATE VIEW | 뷰를 만드는 명령(`CREATE VIEW 이름 AS SELECT ...`) | ⑧ 뷰 | DROP VIEW |
| DROP VIEW IF EXISTS | 뷰가 있으면 지우는 명령(SQLite에서 재정의 시 사용) | ⑧ 뷰 | CREATE VIEW |
| sqlite3 | Python 표준 라이브러리로 설치 없이 SQLite를 다루는 모듈 | ⑨ Python 연동 | connect, cursor |
| connect() | DB 파일에 접속하는 함수 | ⑨ Python 연동 | cursor, close |
| cursor / execute | 쿼리를 보내고 결과를 다루는 객체와 그 실행 메서드 | ⑨ Python 연동 | fetchall |
| fetchall() | 쿼리 결과의 모든 행을 한 번에 받아오는 메서드 | ⑨ Python 연동 | fetchone, cursor |
| commit() | 데이터 변경(INSERT/UPDATE/DELETE)을 확정하는 메서드 | ⑨ Python 연동 | CRUD |
| CRUD | 생성·조회·수정·삭제(Create·Read·Update·Delete)의 묶음 | ⑨ Python 연동 | commit, SQL |
| matplotlib | Python 대표 시각화 라이브러리(`fig, ax = plt.subplots()`) | ⑨ Python 연동 | 막대그래프, 시각화 |
| row_factory | 결과 행을 컬럼명으로 접근하게 해 주는 설정(`sqlite3.Row`) | ⑨ Python 연동 | fetchall |
