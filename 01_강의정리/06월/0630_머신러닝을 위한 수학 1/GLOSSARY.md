# 용어집

이번 강의에서 등장한 핵심 용어를 쉬운 말로 정리했다. 흐름은 선형대수 큰 그림 → 벡터 → 행렬 → 연립방정식 → Python 실습 → 데이터 결합 순서다.

## 선형대수 큰 그림

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 선형대수학 | 데이터를 벡터와 행렬로 표현하고 계산하는 수학 분야. 머신러닝 모델의 입력, 계산, 압축을 이해하는 언어다. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 벡터, 행렬, 차원 축소 |
| 벡터 | 숫자를 한 줄로 나열한 데이터 표현. 크기와 방향을 가진 화살표로 이해할 수 있다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 내적, 노름, 코사인 유사도 |
| 행렬 | 숫자를 행과 열로 배열한 표. 여러 벡터를 묶거나 데이터셋 전체를 표현할 때 사용한다. | [행렬과 행렬 연산](03-matrix-operations.md) | 행렬 곱, 전치, 역행렬 |
| 텐서 | 3차원 이상의 숫자 묶음. 컬러 이미지는 높이×너비×채널 형태의 텐서로 볼 수 있다. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 이미지 데이터, 행렬 |
| Bag-of-Words | 문장에 단어가 등장했는지를 0과 1로 표시해 문장을 벡터로 바꾸는 방식. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 텍스트 벡터화, 임베딩 |
| 신경망 선형층 | 입력 벡터에 가중치 행렬을 곱하고 편향을 더하는 계산. 보통 `u = Wx + b`로 표현한다. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 행렬 곱, 활성화 함수 |
| 활성화 함수 | 선형 계산 결과에 비선형성을 추가하는 함수. ReLU, Sigmoid 등이 대표적이다. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 신경망, 비선형성 |
| PCA | 데이터의 정보가 많이 담긴 방향을 찾아 차원을 줄이는 방법. 시각화와 압축에 자주 사용된다. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | 차원 축소, SVD |
| SVD | 행렬을 여러 성분으로 분해해 압축, 추천 시스템, 차원 축소에 활용하는 방법. | [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) | PCA, 행렬 분해 |

## 벡터와 벡터공간

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 벡터공간 $R^n$ | n개의 실수로 이루어진 벡터들이 덧셈과 스칼라곱 규칙을 만족하는 공간. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 스칼라곱, 차원 |
| 차원 | 벡터에 들어 있는 숫자의 개수. 3차원 벡터는 숫자 3개짜리 벡터다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | $R^n$, 고차원 데이터 |
| 스칼라 | 방향이 없는 하나의 숫자. 벡터에 곱하면 크기나 방향이 바뀐다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 스칼라곱, 벡터 |
| 내적 | 두 벡터의 같은 위치 성분을 곱해 모두 더한 값. 결과는 하나의 숫자다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 코사인 유사도, 직교 |
| 노름 | 벡터의 길이 또는 크기. 기본적으로 L2 노름을 자주 쓴다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 거리, L1/L2/L∞ |
| 거리 | 두 벡터 사이의 떨어진 정도. 보통 두 벡터의 차 벡터의 노름으로 계산한다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 노름, 유클리디안 거리 |
| 직교 | 두 벡터가 서로 수직인 상태. 내적이 0이면 직교다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 내적, 각도 |
| 코사인 유사도 | 두 벡터의 크기를 제외하고 방향이 얼마나 비슷한지 비교하는 지표. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 내적, 노름, 추천 시스템 |
| L1 노름 | 성분 절댓값의 합. 격자형 이동 거리처럼 이해할 수 있다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 맨해튼 거리, L2 노름 |
| L2 노름 | 성분 제곱합의 제곱근. 직선 거리와 연결된다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 유클리디안 거리, 피타고라스 |
| L∞ 노름 | 성분 차이 중 가장 큰 값을 보는 방식. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 노름, 거리 |
| 코시-슈바르츠 부등식 | 내적의 절댓값은 두 벡터 노름의 곱보다 클 수 없다는 부등식. 코사인 유사도의 범위를 설명한다. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 내적, 노름 |
| 삼각 부등식 | 두 점 사이의 직선 거리는 우회 거리보다 짧거나 같다는 성질. | [벡터와 벡터공간](02-vectors-and-vector-space.md) | 거리, 노름 |

## 행렬과 행렬 연산

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 정방행렬 | 행 수와 열 수가 같은 행렬. 역행렬과 행렬식은 정방행렬에서 중요하다. | [행렬과 행렬 연산](03-matrix-operations.md) | 역행렬, 행렬식 |
| 행벡터 | 1행짜리 행렬. 가로 방향으로 놓인 벡터다. | [행렬과 행렬 연산](03-matrix-operations.md) | 열벡터, 전치 |
| 열벡터 | 1열짜리 행렬. 세로 방향으로 놓인 벡터다. | [행렬과 행렬 연산](03-matrix-operations.md) | 행벡터, 전치 |
| 행렬 덧셈 | 같은 크기의 행렬끼리 같은 위치 성분을 더하는 연산. | [행렬과 행렬 연산](03-matrix-operations.md) | 행렬 크기, 스칼라곱 |
| 행렬 곱 | 왼쪽 행렬의 행과 오른쪽 행렬의 열을 내적해 새 행렬을 만드는 연산. | [행렬과 행렬 연산](03-matrix-operations.md) | 내적, 행렬 크기 |
| 전치행렬 | 행과 열을 서로 바꾼 행렬. `A.T`로 계산할 수 있다. | [행렬과 행렬 연산](03-matrix-operations.md) | 대칭행렬, 행벡터 |
| 대칭행렬 | 전치해도 자기 자신과 같은 행렬. 대각선을 기준으로 위아래가 같다. | [행렬과 행렬 연산](03-matrix-operations.md) | 전치행렬, 공분산 행렬 |
| 단위행렬 | 대각선은 1, 나머지는 0인 행렬. 행렬 곱셈에서 숫자 1 같은 역할을 한다. | [행렬과 행렬 연산](03-matrix-operations.md) | 역행렬, 행렬 곱 |
| 역행렬 | 어떤 행렬과 곱했을 때 단위행렬이 되는 행렬. 변환을 되돌리는 역할을 한다. | [행렬과 행렬 연산](03-matrix-operations.md) | 단위행렬, 행렬식 |
| 행렬식 | 정방행렬에 대응하는 하나의 숫자. 0이면 역행렬이 없다. | [행렬과 행렬 연산](03-matrix-operations.md) | 역행렬, 특이행렬 |
| 특이행렬 | 행렬식이 0이라 역행렬이 존재하지 않는 행렬. | [행렬과 행렬 연산](03-matrix-operations.md) | 행렬식, 선형종속 |
| 선형변환 | 벡터를 다른 벡터로 바꾸는 규칙. 행렬은 선형변환을 표현하는 도구다. | [행렬과 행렬 연산](03-matrix-operations.md) | 행렬 곱, 좌표 변환 |

## 연립방정식과 소거법

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 연립방정식 | 여러 방정식을 동시에 만족하는 해를 찾는 문제. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | $Ax=b$, 교점 |
| $Ax=b$ | 계수행렬 A, 미지수 벡터 x, 상수 벡터 b로 연립방정식을 표현한 형태. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | 계수행렬, 상수벡터 |
| 계수행렬 | 미지수 앞에 붙은 숫자만 모은 행렬. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | 첨가행렬, $Ax=b$ |
| 첨가행렬 | 계수행렬 A 오른쪽에 상수 벡터 b를 붙인 행렬. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | Gauss 소거법 |
| Gauss 소거법 | 첨가행렬을 상삼각행렬로 만든 뒤 후진대입으로 해를 구하는 방법. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | 후진대입, 상삼각행렬 |
| Gauss-Jordan 소거법 | 왼쪽을 단위행렬까지 만들어 해를 직접 읽는 방법. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | RREF, 단위행렬 |
| 후진대입 | 상삼각행렬의 아래쪽 식부터 위로 올라가며 미지수 값을 구하는 과정. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | Gauss 소거법 |
| 유일해 | 연립방정식을 만족하는 해가 하나만 있는 경우. 보통 계수행렬의 행렬식이 0이 아닐 때 나타난다. | [연립방정식과 가우스 소거법](04-linear-equations-gauss.md) | 행렬식, 역행렬 |

## Python 실습

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `np.dot` | 벡터 내적 또는 행렬 곱에 사용할 수 있는 NumPy 함수. | [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) | 내적, 행렬 곱 |
| `@` 연산자 | Python에서 행렬 곱셈을 표현하는 연산자. | [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) | 행렬 곱, 내적 |
| `np.linalg.norm` | 벡터나 행렬의 노름을 계산하는 함수. 기본적으로 벡터의 L2 노름을 구한다. | [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) | 노름, 거리 |
| `np.linalg.solve` | 연립방정식 $Ax=b$의 해를 구하는 함수. | [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) | $Ax=b$, 연립방정식 |
| 해 검증 | 구한 해 x를 다시 `A @ x`에 넣어 `b`가 나오는지 확인하는 과정. | [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) | `np.linalg.solve`, 행렬 곱 |

## 데이터 결합

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 피처 테이블 | 머신러닝 모델에 넣기 위해 정리한 최종 데이터 표. 보통 한 행은 관측치, 한 열은 변수다. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | feature, matrix X |
| `concat` | 같은 구조의 데이터를 위아래 또는 좌우로 단순히 이어 붙이는 함수. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | axis, ignore_index |
| `merge` | 공통 키를 기준으로 두 DataFrame을 결합하는 함수. SQL JOIN과 비슷하다. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | inner, left, right, outer |
| inner join | 양쪽 테이블에 모두 존재하는 키만 남기는 조인. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | left join, outer join |
| left join | 왼쪽 테이블의 행을 모두 유지하고 오른쪽 정보를 붙이는 조인. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | inner join, NaN |
| outer join | 양쪽 테이블의 모든 키를 보존하는 조인. 매칭되지 않는 곳은 NaN이 생긴다. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | 결측치, NaN |
| `join` | 인덱스를 기준으로 DataFrame을 결합하는 메서드. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | set_index, merge |
| `merge_asof` | 정확히 같은 시간이 없어도 가장 가까운 이전 또는 이후 데이터를 붙이는 시계열 병합 함수. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | 시계열, 정렬 |
| `combine_first` | 첫 번째 데이터의 결측값만 두 번째 데이터로 보완하는 메서드. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | 결측치 처리 |
| `compare` | 두 DataFrame에서 달라진 값만 비교해 보여주는 메서드. | [데이터 결합으로 피처 테이블 만들기](06-pandas-merging-feature-table.md) | 데이터 검증, 변경 확인 |
