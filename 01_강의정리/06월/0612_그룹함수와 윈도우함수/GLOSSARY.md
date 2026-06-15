# 용어집 (GLOSSARY)

이 강의(윈도우 함수 & 그룹 함수)에서 등장한 핵심 용어를 쉬운 말로 정리했습니다.

| 용어 | 쉬운 설명 | 관련 강의명 | 함께 보면 좋은 개념 |
| ---- | --------- | ----------- | ----------- |
| 윈도우 함수 | 결과의 행 수를 줄이지 않고, 행과 행의 관계(순위·집계·비교)를 계산해 옆에 붙이는 함수 | 윈도우 함수 첫걸음 | 집계 함수, OVER |
| 집계 함수 | 여러 행을 하나로 합치는 함수(SUM·AVG·COUNT·MAX·MIN). GROUP BY와 함께 쓰면 행이 압축됨 | 윈도우 함수 첫걸음 | GROUP BY, 윈도우 함수 |
| 그룹 함수 | GROUP BY 결과에 소계·중계·총계를 자동으로 덧붙이는 함수(ROLLUP·CUBE 등) | 그룹 함수 | GROUP BY, ROLLUP |
| OVER | 윈도우 함수에 반드시 따라오는 구문. "어떤 범위(창)를 기준으로 계산할지"를 지정 | 윈도우 함수 첫걸음 | PARTITION BY, ORDER BY |
| PARTITION BY | 전체 데이터를 소그룹(칸막이)으로 나누는 기준. 그룹별 순위·집계를 만들 때 사용 | 윈도우 함수 첫걸음 | GROUP BY, ORDER BY |
| ORDER BY (윈도우) | 창 안에서의 정렬 기준. 순위·누적·LAG/LEAD의 순서를 결정(출력용 ORDER BY와 별개) | 순위 함수 3형제 | RANK, LAG |
| WINDOWING / 프레임 | 한 행을 계산할 때 "어디부터 어디까지"를 한 묶음으로 볼지 정하는 범위 절 | 집계 윈도우 함수와 누적합 | ROWS BETWEEN, 누적합 |
| ROWS BETWEEN | 물리적 행 단위로 창의 시작·끝 경계를 지정하는 문법 | 집계 윈도우 함수와 누적합 | UNBOUNDED PRECEDING, CURRENT ROW |
| UNBOUNDED PRECEDING | 창의 시작을 "첫 번째 행"으로 여는 키워드 | 집계 윈도우 함수와 누적합 | 누적합, WINDOWING |
| UNBOUNDED FOLLOWING | 창의 끝을 "마지막 행"으로 여는 키워드 | 행 순서 함수 | LAST_VALUE, WINDOWING |
| CURRENT ROW | 창의 경계를 "현재 행"에 두는 키워드 | 집계 윈도우 함수와 누적합 | 누적합, ROWS BETWEEN |
| RANK | 동점은 같은 순위, 그 수만큼 다음 순위를 건너뛰는 순위 함수(1,2,3,3,5) | 순위 함수 3형제 | DENSE_RANK, ROW_NUMBER |
| DENSE_RANK | 동점은 같은 순위, 다음 순위를 건너뛰지 않는 순위 함수(1,2,3,3,4) | 순위 함수 3형제 | RANK, 등급 |
| ROW_NUMBER | 동점이어도 무조건 고유한 연속 번호를 매기는 함수(1,2,3,4,5) | 순위 함수 3형제 | RANK, 그룹별 1행 추출 |
| 누적합 (러닝 토탈) | 정렬 순서대로 첫 행부터 현재 행까지 더해 가는 합계 | 집계 윈도우 함수와 누적합 | ROWS BETWEEN, SUM OVER |
| FIRST_VALUE | 정렬된 창에서 가장 먼저 나오는 값(그룹 최솟값 등) | 행 순서 함수 | LAST_VALUE, PARTITION BY |
| LAST_VALUE | 정렬된 창에서 가장 나중에 나오는 값. 범위를 전체로 열어야 의도대로 동작 | 행 순서 함수 | FIRST_VALUE, UNBOUNDED FOLLOWING |
| LAG | 현재 행 기준 N칸 이전 행의 값을 가져옴(첫 행은 NULL) | 행 순서 함수 | LEAD, 전월 대비 |
| LEAD | 현재 행 기준 N칸 이후 행의 값을 가져옴(마지막 행은 NULL) | 행 순서 함수 | LAG, 시계열 분석 |
| 전월 대비 (MoM) | 이번 달 값에서 직전 달 값을 빼/나눠 증감액·증감률을 구하는 분석 | 행 순서 함수 | LAG, strftime |
| strftime | 날짜에서 연·월 등을 문자열로 추출하는 함수(예: '%Y-%m') | 행 순서 함수 | julianday, GROUP BY |
| julianday | 날짜를 숫자(일 단위)로 바꿔 두 날짜의 차이를 뺄셈으로 계산하게 해 주는 함수 | 행 순서 함수 | 재주문 간격, 이탈 분석 |
| 이탈(Churn) 분석 | 평소 주기보다 오래 주문/활동이 없는 고객을 가려내는 분석 | 행 순서 함수 | LAG, 재주문 간격 |
| RATIO_TO_REPORT | 파티션 전체 합에서 각 행이 차지하는 비율. 미지원 DB는 값/SUM() OVER()로 대체 | 비율과 등급 | SUM OVER, 비중 분석 |
| PERCENT_RANK | 순위를 0~1 백분율로 표현(최고 순위=0, 최저=1) | 비율과 등급 | CUME_DIST, 순위 |
| CUME_DIST | 현재 행을 포함한 누적 분포 백분율(0 초과~1) | 비율과 등급 | PERCENT_RANK, 분위 |
| NTILE | 정렬된 행을 N등분해 등급 번호(1~N)를 부여 | 비율과 등급 | 등급 분류, PARTITION BY |
| GROUP BY | 지정한 컬럼이 같은 행끼리 묶어 집계하는 구문(행이 압축됨) | 그룹 함수 | 집계 함수, ROLLUP |
| ROLLUP | (A,B) 상세 + (A) 소계 + 전체 총계까지 계층적으로 만드는 그룹 함수 | 그룹 함수 | CUBE, GROUPING SETS |
| CUBE | ROLLUP에 더해 모든 컬럼 조합의 소계·총계를 만드는 그룹 함수 | 그룹 함수 | ROLLUP, 다차원 집계 |
| GROUPING SETS | 지정한 컬럼들 각각의 통계만 골라 만드는 그룹 함수 | 그룹 함수 | ROLLUP, UNION ALL |
| UNION ALL | 여러 SELECT 결과를 그대로 이어 붙이는 연산(중복 제거 안 함). 미지원 그룹 함수 대체에 사용 | 그룹 함수 | UNION, GROUPING SETS |
| 스칼라 서브쿼리 | 단일 값 하나를 돌려주는 서브쿼리. 전체 합을 분모로 따로 구할 때 유용 | 집계 윈도우 함수와 누적합 | 비중 분석, SUM OVER |
| 인라인 뷰 | FROM 절 안에 들어가는 서브쿼리. 윈도우 함수 결과를 WHERE로 거를 때 감싸는 용도 | 순위 함수 3형제 | 그룹별 1행 추출, 서브쿼리 |
