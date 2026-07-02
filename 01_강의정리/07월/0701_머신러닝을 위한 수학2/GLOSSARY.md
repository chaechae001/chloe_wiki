# 용어 사전 (GLOSSARY)

머신러닝을 위한 수학 — 선형대수학과 pandas 시계열 처리에서 등장한 핵심 용어를 한눈에 정리했습니다. 처음 보는 용어가 나오면 여기서 빠르게 찾아보세요. "관련 강의명"을 누르면 해당 개념을 자세히 다룬 포스트로 이어집니다.

| 용어 | 쉬운 설명 | 관련 강의명 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| 벡터공간 (Vector Space) | 벡터끼리 더하거나 상수배를 해도 그 공간을 벗어나지 않는, "닫혀 있는" 벡터들의 세계 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 부분공간, 일차결합 |
| 부분공간 (Subspace) | 벡터공간 안에 들어 있는 더 작은 벡터공간. 원점을 반드시 포함하고 덧셈·상수배에 닫혀 있어야 함 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 벡터공간, Span |
| 일차결합 (Linear Combination) | 여러 벡터에 상수를 곱해 더한 것. 예: 2u + 3v | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | Span, 일차독립 |
| Span (생성) | 주어진 벡터들의 모든 일차결합을 모아 만든 공간 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 일차결합, 기저 |
| 일차독립 (Linearly Independent) | 어떤 벡터도 나머지 벡터들의 조합으로 만들 수 없는 상태. 서로 "겹치는 정보"가 없음 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 일차종속, 기저 |
| 일차종속 (Linearly Dependent) | 한 벡터가 다른 벡터들의 조합으로 표현되어 정보가 중복되는 상태 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 일차독립, rank |
| 기저 (Basis) | 공간을 빠짐없이·겹침없이 만들어내는 최소한의 벡터 집합. 공간의 "골조" | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 차원, Span |
| 차원 (Dimension) | 기저를 이루는 벡터의 개수. 그 공간을 표현하는 데 필요한 축의 수 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 기저, rank |
| 직교 (Orthogonal) | 두 벡터의 내적이 0인 상태. 서로 90도로 만나 완전히 독립적인 방향 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 내적, 정규직교기저 |
| 정규직교기저 (Orthonormal Basis) | 서로 직교하면서 길이가 모두 1인 기저. 계산이 가장 편한 이상적인 좌표축 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 직교, Gram-Schmidt |
| 그람-슈미트 과정 (Gram-Schmidt) | 아무렇게나 놓인 기저를 서로 직교하는 기저로 다듬는 절차 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 정규직교기저, 직교여공간 |
| 직교여공간 (Orthogonal Complement, W⊥) | 어떤 부분공간 W의 모든 벡터와 직교하는 벡터들을 모은 공간 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 직교, 부분공간 |
| 내적 (Inner Product / Dot Product) | 두 벡터를 곱해 하나의 숫자로 만든 값. 방향의 닮은 정도를 측정 | [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md) | 직교, Norm |
| 선형변환 (Linear Transformation) | 덧셈과 상수배 구조를 그대로 보존하는 변환. 직선을 직선으로, 원점을 원점으로 보냄 | [선형변환과 행렬](posts/02-linear-transformation.md) | 행렬, 표준행렬 |
| 표준행렬 (Standard Matrix) | 선형변환을 나타내는 행렬. 변환을 곱셈 한 번으로 계산하게 해줌 | [선형변환과 행렬](posts/02-linear-transformation.md) | 선형변환, 행렬곱 |
| 전치행렬 (Transpose, Aᵀ) | 행과 열을 뒤바꾼 행렬 | [선형변환과 행렬](posts/02-linear-transformation.md) | 대칭행렬, 직교행렬 |
| 대칭행렬 (Symmetric Matrix) | 전치해도 자기 자신과 같은 행렬(A = Aᵀ) | [선형변환과 행렬](posts/02-linear-transformation.md) | 전치행렬, 대각화 |
| 직교행렬 (Orthogonal Matrix) | 열들이 서로 직교하고 길이가 1인 행렬. QᵀQ = I를 만족하며 회전·반사를 표현 | [선형변환과 행렬](posts/02-linear-transformation.md) | 회전행렬, 정규직교기저 |
| 회전행렬 (Rotation Matrix) | 벡터를 원점 기준으로 일정 각도만큼 돌리는 직교행렬 | [선형변환과 행렬](posts/02-linear-transformation.md) | 직교행렬, 선형변환 |
| 프로베니우스 노름 (Frobenius Norm) | 행렬의 모든 원소를 제곱해 더한 뒤 제곱근을 취한 값. 행렬의 "크기" | [선형변환과 행렬](posts/02-linear-transformation.md) | Norm, 특이값 |
| 역행렬 (Inverse Matrix, A⁻¹) | 곱하면 단위행렬이 되는 행렬. 변환을 원래대로 되돌림 | [선형변환과 행렬](posts/02-linear-transformation.md) | 행렬식, 단위행렬 |
| 단위행렬 (Identity Matrix, I) | 대각선이 1이고 나머지가 0인 행렬. 곱셈에서 "1"의 역할 | [선형변환과 행렬](posts/02-linear-transformation.md) | 역행렬 |
| 행렬식 (Determinant, det) | 행렬이 공간의 넓이·부피를 몇 배로 키우는지 나타내는 수. 0이면 역행렬이 없음 | [선형변환과 행렬](posts/02-linear-transformation.md) | 역행렬, rank |
| 계수 (Rank) | 행렬이 담고 있는 "독립적인 정보"의 개수. 일차독립인 행(또는 열)의 수 | [선형변환과 행렬](posts/02-linear-transformation.md) | 일차독립, 차원 |
| 고윳값 (Eigenvalue, λ) | 선형변환이 특정 방향을 몇 배로 늘리거나 줄이는지 나타내는 스칼라 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고유벡터, 대각화 |
| 고유벡터 (Eigenvector) | 변환을 거쳐도 방향이 바뀌지 않고 크기만 변하는 특별한 벡터 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고윳값, 대각화 |
| 특성방정식 (Characteristic Equation) | det(A − λI) = 0. 이 방정식을 풀면 고윳값이 나옴 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고윳값, 행렬식 |
| 대각화 (Diagonalization) | 행렬을 A = PDP⁻¹ 형태로 분해하는 것. 반복 계산(거듭제곱)을 매우 쉽게 만듦 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고윳값, 고유벡터 |
| 대각행렬 (Diagonal Matrix, D) | 대각선에만 값이 있고 나머지는 0인 행렬 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 대각화, 단위행렬 |
| 스펙트럴 정리 (Spectral Theorem) | 대칭행렬은 항상 직교하는 고유벡터로 대각화된다는 정리 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 대칭행렬, 대각화 |
| 양의 정부호 행렬 (Positive Definite Matrix, PDM) | 모든 고윳값이 양수인 대칭행렬. 최적화·공분산에서 중요 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고윳값, 대칭행렬 |
| 스펙트럴 클러스터링 (Spectral Clustering) | 그래프의 라플라시안 고유벡터를 이용해 데이터를 묶는 군집화 기법 | [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md) | 고유벡터, Fiedler 벡터 |
| 특이값 분해 (SVD) | 모든 행렬을 A = UΣVᵀ로 쪼개는 만능 분해. 정사각형이 아니어도 항상 가능 | [특이값 분해 (SVD)](posts/04-singular-value-decomposition.md) | 특이값, rank-k 근사 |
| 특이값 (Singular Value, σ) | Σ의 대각 성분. 각 방향으로 데이터가 얼마나 뻗어 있는지 나타내는 크기 | [특이값 분해 (SVD)](posts/04-singular-value-decomposition.md) | SVD, 고윳값 |
| rank-k 근사 (Low-Rank Approximation) | 큰 특이값 k개만 남겨 행렬을 근사하는 것. 데이터·이미지 압축의 핵심 | [특이값 분해 (SVD)](posts/04-singular-value-decomposition.md) | 특이값, 이미지 압축 |
| 이미지 압축 (Image Compression) | 이미지를 행렬로 보고 SVD로 상위 특이값만 남겨 용량을 줄이는 기법 | [특이값 분해 (SVD)](posts/04-singular-value-decomposition.md) | rank-k 근사, 특이값 |
| Timestamp | pandas가 특정 "순간"을 표현하는 자료형. 날짜·시간 데이터의 기본 단위 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | DatetimeIndex, dt 접근자 |
| DatetimeIndex | Timestamp들로 이루어진 인덱스. 날짜 기반 슬라이싱·리샘플링을 가능하게 함 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | Timestamp, resample |
| dt 접근자 (dt accessor) | 날짜 컬럼에서 연·월·일·요일 등을 뽑아내는 pandas 기능(예: df['날짜'].dt.year) | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | Timestamp, DatetimeIndex |
| Timedelta | 두 시점 사이의 "시간 간격"을 표현하는 자료형 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | Timestamp, DateOffset |
| DateOffset | 달력 규칙(월말·다음 달 등)을 반영해 날짜를 이동시키는 객체 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | Timedelta, resample |
| resample | 시계열 데이터를 일·월·주 등 원하는 주기로 다시 묶어 집계하는 기능 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | DatetimeIndex, DateOffset |
| shift | 데이터를 시간 축으로 밀어 이전·다음 값을 가져오는 기능. 증감률 계산에 사용 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | rolling, resample |
| rolling | 이동창(window)을 미끄러뜨리며 이동평균 등을 계산하는 기능 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | shift, resample |
| tz_localize | 시간대 정보가 없는 시각에 시간대를 부여하는 기능 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | tz_convert, Timestamp |
| tz_convert | 이미 시간대가 있는 시각을 다른 시간대로 변환하는 기능 | [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md) | tz_localize, Timestamp |

---

## 함께 보기

- [강의 개요 (OVERVIEW)](OVERVIEW.md) — 전체 주제 흐름과 학습 로드맵
- [벡터공간과 부분공간](posts/01-vector-space-and-subspace.md)
- [선형변환과 행렬](posts/02-linear-transformation.md)
- [고윳값·고유벡터·행렬의 대각화](posts/03-eigenvalue-eigenvector-diagonalization.md)
- [특이값 분해 (SVD)](posts/04-singular-value-decomposition.md)
- [pandas 시계열 처리 50제](posts/05-pandas-timeseries.md)
