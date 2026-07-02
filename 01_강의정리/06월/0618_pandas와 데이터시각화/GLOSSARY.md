# 📖 용어집 (Glossary)

Pandas 데이터 분석에서 자주 쓰는 핵심 용어를 쉬운 말로 정리했습니다.

| 용어 | 쉬운 설명 | 관련 강의명 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| DataFrame | 행과 열로 이루어진 2차원 표. 엑셀 시트 한 장 같은 것 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | Series, read_csv |
| Series | 표에서 떼어낸 컬럼 한 줄(1차원). 인덱스+값으로 구성 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | DataFrame, value_counts |
| read_csv / read_excel | 파일로 저장된 데이터를 표(DataFrame)로 불러오는 함수 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | DataFrame, 경로 |
| info() | 컬럼 이름·자료형·결측치 개수를 한눈에 보여 주는 점검 도구 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | dtype, describe |
| describe() | 수치형 컬럼의 평균·최소·최대·사분위수 등 통계 요약 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | mean, groupby |
| dtype | 컬럼의 자료형(int/float/bool/datetime/category/object) | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | astype, to_numeric |
| value_counts() | 한 컬럼에서 각 값이 몇 번 나오는지 빈도를 세는 함수 | Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기 | unique, groupby |
| unique() | 컬럼에 어떤 값들이 있는지 종류만 확인(SQL의 DISTINCT) | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | value_counts |
| loc | 라벨(인덱스·컬럼 이름) 기반으로 데이터를 고르는 방법 | loc와 iloc — 라벨로 뽑을까, 위치로 뽑을까 | iloc, Boolean indexing |
| iloc | 위치(정수 순서) 기반으로 데이터를 고르는 방법 | loc와 iloc — 라벨로 뽑을까, 위치로 뽑을까 | loc, 슬라이싱 |
| 슬라이싱 끝값 미포함 | `iloc[0:10]`은 0~9까지만(끝 번호 10 제외) | loc와 iloc — 라벨로 뽑을까, 위치로 뽑을까 | iloc |
| copy() | 데이터 일부를 원본과 분리해 복사(원본 보호) | loc와 iloc — 라벨로 뽑을까, 위치로 뽑을까 | reset_index |
| Boolean indexing | 조건의 참/거짓(True/False)으로 원하는 행만 남기는 필터링 | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | loc, isin, between |
| 비트 연산자(&, \|, ~) | 여러 조건을 결합(그리고/또는/부정). 각 조건은 괄호 필수 | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | Boolean indexing |
| isin() | 여러 후보 값 중 하나에 해당하는지 검사(SQL의 IN) | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | between, ~ |
| between() | 수치가 특정 구간(하한~상한)에 드는지 검사 | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | isin |
| reset_index(drop=True) | 뒤섞인 인덱스를 0부터 다시 정리(기존 인덱스는 버림) | 조건 필터링(Boolean indexing) — 원하는 행만 골라내기 | copy, 필터링 |
| astype() | 컬럼의 자료형을 원하는 타입으로 변환 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | to_numeric, dtype |
| to_numeric() | 컬럼 값을 숫자 타입으로 변환 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | astype |
| to_datetime() | 문자열 날짜를 계산 가능한 시간 타입으로 변환 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | dt |
| dt 접근자 | 시간 타입에서 연·월·일·요일을 꺼내는 도구(dt.year 등) | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | to_datetime, map |
| map() | 컬럼 값을 규칙(딕셔너리 등)대로 한 번에 치환 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | apply, lambda |
| apply() | 컬럼/행에 함수를 적용(복수 컬럼 가능, axis로 방향 지정) | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | map, lambda |
| lambda | 콜론 왼쪽 입력, 오른쪽 반환값으로 함수를 한 줄로 쓰는 문법 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | apply |
| sort_values() | 특정 컬럼 값 기준으로 전체 데이터를 정렬(ascending 옵션) | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | reset_index |
| drop() | 특정 행(axis=0)이나 열(axis=1)을 삭제 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | rename |
| rename() | 컬럼 이름을 바꿈(columns 딕셔너리 사용) | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | drop |
| isnull() / isna() | 각 값이 결측치(NaN)인지 True/False로 표시 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | fillna, dropna |
| fillna() | 결측치를 다른 값(평균·중앙값·최빈값 등)으로 채움 | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | isnull, dropna |
| dropna() | 결측치가 있는 행을 삭제(subset·ignore_index 옵션) | 데이터 변환·정제 — 타입 바꾸고 결측치 다루기 | fillna |
| groupby() | 기준 컬럼으로 나누고 집계한 뒤 합치는 그룹 요약 | 데이터 요약·병합 — groupby·concat·merge | agg, unstack |
| Split-Apply-Combine | groupby의 동작 원리: 나누고 → 적용하고 → 합친다 | 데이터 요약·병합 — groupby·concat·merge | groupby |
| agg() | 한 그룹화 결과에 여러 집계 함수를 동시에 적용 | 데이터 요약·병합 — groupby·concat·merge | groupby, unstack |
| unstack() | 다중 인덱스 결과를 표(피벗) 형태로 펼쳐 보기 좋게 변환 | 데이터 요약·병합 — groupby·concat·merge | groupby, agg |
| concat() | 구조가 같은 표를 위아래/좌우로 이어 붙임 | 데이터 요약·병합 — groupby·concat·merge | merge |
| merge() | 공통 컬럼(key)을 기준으로 두 표를 연결(SQL의 JOIN) | 데이터 요약·병합 — groupby·concat·merge | concat, how |
| how (inner/outer/left/right) | merge에서 어느 쪽 행을 남길지 정하는 방식 | 데이터 요약·병합 — groupby·concat·merge | merge |
