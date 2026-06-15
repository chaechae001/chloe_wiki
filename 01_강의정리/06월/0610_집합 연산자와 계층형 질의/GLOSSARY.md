# 용어집 (GLOSSARY)

이번 강의(집합 연산자 · 계층형 질의)에서 등장한 용어를 쉬운 말로 정리했습니다.
"관련 글"의 제목을 누르면 해당 글로 이동합니다.

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| 관계형 대수 | DB에서 원하는 정보를 끌어내는 기본 연산들의 모음. SQL의 뼈대 | [관계형 대수](posts/01-relational-algebra.md) | 집합 연산, 관계 연산 |
| 일반 집합 연산 | 두 표를 통째로 합치거나 빼는 연산(합·교·차집합, 카티션 곱) | [관계형 대수](posts/01-relational-algebra.md) | UNION, INTERSECT, EXCEPT |
| 순수 관계 연산 | 한 표를 다듬고 다른 표와 잇는 연산(셀렉션·프로젝션·조인·디비전) | [관계형 대수](posts/01-relational-algebra.md) | WHERE, SELECT, JOIN |
| 셀렉션(Selection) | 조건에 맞는 **행**을 고르는 연산. SQL의 `WHERE`에 대응 | [관계형 대수](posts/01-relational-algebra.md) | 프로젝션, WHERE |
| 프로젝션(Projection) | 원하는 **열**만 추리는 연산. SQL의 `SELECT` 절에 대응 | [관계형 대수](posts/01-relational-algebra.md) | 셀렉션, SELECT |
| 카티션 곱 | 두 표의 모든 행 조합을 만드는 연산. 행 수가 폭발적으로 늘어남 | [관계형 대수](posts/01-relational-algebra.md) | CROSS JOIN, 조인 |
| 집계 함수 | 여러 행을 입력받아 단일 값 하나를 돌려주는 함수 | [집계 함수](posts/02-aggregate-functions.md) | GROUP BY, HAVING |
| COUNT | 행의 개수를 세는 집계 함수 | [집계 함수](posts/02-aggregate-functions.md) | DISTINCT, COUNT(*) |
| SUM / AVG / MAX / MIN | 합·평균·최댓값·최솟값을 구하는 집계 함수 | [집계 함수](posts/02-aggregate-functions.md) | GROUP BY, ROUND |
| DISTINCT | 중복을 제거한 뒤 처리. `COUNT(DISTINCT ...)`가 대표적 | [집계 함수](posts/02-aggregate-functions.md) | COUNT, 고유 값 |
| GROUP BY | 같은 값을 가진 행을 그룹으로 묶어 그룹별로 집계 | [집계 함수](posts/02-aggregate-functions.md) | 집계 함수, HAVING |
| HAVING | 집계 **후** 그룹에 거는 조건. `WHERE`는 집계 전 행에 걸림 | [집계 함수](posts/02-aggregate-functions.md) | WHERE, GROUP BY |
| GROUP_CONCAT | 같은 그룹의 여러 행 값을 하나의 문자열로 이어 붙임 | [집계 함수](posts/02-aggregate-functions.md) | DISTINCT, 구분자 |
| strftime | 날짜에서 연도·월 등을 뽑아내는 SQLite 함수(예: `%Y`=연도) | [집계 함수](posts/02-aggregate-functions.md) | GROUP BY, 날짜 처리 |
| 서브쿼리 | 다른 SQL 문 안에 들어 있는 `SELECT`. "쿼리 속의 쿼리" | [서브쿼리](posts/03-subquery.md) | 인라인 뷰, 스칼라 서브쿼리 |
| 인라인 뷰 | `FROM` 절에 들어가 임시 테이블처럼 쓰이는 서브쿼리 | [서브쿼리](posts/03-subquery.md) | 파생 테이블, 서브쿼리 |
| 스칼라 서브쿼리 | `SELECT` 절에 들어가 행마다 단일 값 하나를 돌려주는 서브쿼리 | [서브쿼리](posts/03-subquery.md) | 연관 서브쿼리 |
| 연관 서브쿼리 | 바깥 쿼리의 값을 참조해 행마다 다시 실행되는 서브쿼리 | [서브쿼리](posts/03-subquery.md) | 비연관 서브쿼리, 성능 |
| 비연관 서브쿼리 | 바깥과 상관없이 한 번만 실행되어 값이 정해지는 서브쿼리 | [서브쿼리](posts/03-subquery.md) | 연관 서브쿼리 |
| IN / NOT IN | 값이 서브쿼리 결과 목록 안에 있는지/없는지로 거름 | [서브쿼리](posts/03-subquery.md) | EXISTS, NULL 함정 |
| EXISTS / NOT EXISTS | 서브쿼리가 행을 하나라도 돌려주는지로 참/거짓 판단 | [서브쿼리](posts/03-subquery.md) | IN, 존재 여부 |
| 별칭(Alias) | 테이블·결과에 짧은 이름을 붙이는 것. 가독성에 사실상 필수 | [서브쿼리](posts/03-subquery.md) | 인라인 뷰, 조인 |
| 집합 연산자 | 조인 없이 두 쿼리 결과를 위아래로 묶는 연산자 | [집합 연산자](posts/04-set-operators.md) | UNION, INTERSECT, EXCEPT |
| UNION | 두 결과를 합치고 **중복을 제거**하는 합집합 | [집합 연산자](posts/04-set-operators.md) | UNION ALL, 정렬 |
| UNION ALL | 합치되 **중복 제거·정렬을 하지 않는** 합집합(더 빠름) | [집합 연산자](posts/04-set-operators.md) | UNION, 소계 행 |
| INTERSECT | 양쪽에 모두 있는 행만 남기는 교집합 | [집합 연산자](posts/04-set-operators.md) | UNION, JOIN |
| EXCEPT (MINUS) | 앞 결과에서 뒤 결과에 있는 행을 빼는 차집합 | [집합 연산자](posts/04-set-operators.md) | 이탈 고객, 순서 주의 |
| 소계 행 | 그룹별 집계 아래에 `UNION ALL`로 덧붙이는 전체 합계 행 | [복합 활용](posts/06-composite-report.md) | UNION ALL, 리포트 |
| 계층형 데이터 | 같은 테이블에 상위·하위가 함께 들어 있는 트리형 데이터 | [계층형 질의](posts/05-hierarchical-query.md) | 순환관계, 루트 |
| 계층형 질의 | 계층형 데이터를 계단처럼 펼쳐 조회하는 기법 | [계층형 질의](posts/05-hierarchical-query.md) | 재귀 CTE, CONNECT BY |
| 루트(Root) | 부모가 없는(NULL) 최상위 행. 계층의 시작점 | [계층형 질의](posts/05-hierarchical-query.md) | 리프, 레벨 |
| 재귀 CTE | 자기 자신을 다시 참조하며 한 단계씩 결과를 쌓는 임시 테이블 | [계층형 질의](posts/05-hierarchical-query.md) | WITH RECURSIVE, 앵커/재귀 |
| WITH RECURSIVE | 재귀 CTE를 시작하는 키워드 | [계층형 질의](posts/05-hierarchical-query.md) | 앵커 쿼리, 재귀 쿼리 |
| 앵커 쿼리 | 재귀 CTE의 시작점(루트)을 만드는 첫 부분 | [계층형 질의](posts/05-hierarchical-query.md) | 재귀 쿼리, UNION ALL |
| 재귀 쿼리 | 직속 자식을 반복 탐색하며 계층을 내려가는 부분 | [계층형 질의](posts/05-hierarchical-query.md) | 앵커 쿼리, 순환 |
| LEVEL | 계층의 깊이. 루트가 1(Oracle) 또는 0(재귀 CTE 예제) | [계층형 질의](posts/05-hierarchical-query.md) | 들여쓰기, 트리 |
| CONNECT BY PRIOR | 상위·하위를 잇는 Oracle 전용 계층형 질의 문법 | [계층형 질의](posts/05-hierarchical-query.md) | START WITH, 재귀 CTE |
| START WITH | 계층의 루트를 지정하는 Oracle 전용 절 | [계층형 질의](posts/05-hierarchical-query.md) | CONNECT BY, 루트 |
| 리프(Leaf) | 더 이상 자식이 없는 끝 노드 | [계층형 질의](posts/05-hierarchical-query.md) | 루트, 경로(path) |
| CTE (Common Table Expression) | `WITH`로 이름을 붙인 임시 결과 테이블 | [계층형 질의](posts/05-hierarchical-query.md) | 인라인 뷰, 재귀 CTE |
