# Pandas로 표 데이터 다루기
> 실제 분석 데이터는 대부분 엑셀처럼 행과 열을 가진 표 형태다. Pandas는 이 표 데이터를 읽고, 선택하고, 정리하는 핵심 도구다.

`Pandas` · `Series` · `DataFrame` · `index` · `columns` · `loc` · `iloc` · `drop` · `sort_values`

## 핵심요약

- Pandas는 표 형태 데이터를 다루기 위한 Python 라이브러리다.
- `Series`는 인덱스가 붙은 1차원 데이터다.
- `DataFrame`은 행과 열을 가진 2차원 표 데이터다.
- Series끼리 연산하면 인덱스 기준으로 값이 정렬되어 계산된다.
- `loc`는 이름 기반 선택, `iloc`는 위치 기반 선택이다.
- `drop()`은 행이나 열을 삭제한다.
- `sort_values()`는 특정 열 기준으로 데이터를 정렬한다.
- 머신러닝 전처리에서는 결측치 처리, 범주형 인코딩, 특성 선택에 Pandas를 많이 사용한다.

## Series

### 개념명

**1. 정의**  
Series는 한 줄짜리 데이터에 인덱스가 붙은 구조다. 엑셀의 한 열 또는 한 행을 생각하면 쉽다.

**2. 왜 필요한가?**  
데이터를 단순히 순서로만 보는 것이 아니라, 이름이나 날짜 같은 인덱스로 관리할 수 있다.

**3. 예시**

```python
import pandas as pd

obj = pd.Series([4, 7, -5])
print(obj)
```

```text
0    4
1    7
2   -5
dtype: int64
```

```python
obj = pd.Series([4, 7, -5], index=['2016-01-01', '2016-01-02', '2016-01-03'])
print(obj)
```

**4. 헷갈리기 쉬운 점**  
Series는 리스트처럼 보이지만, 각 값에 인덱스가 붙어 있다. 그래서 연산할 때 위치가 아니라 인덱스 기준으로 맞춰질 수 있다.

**5. 한 줄 정리**  
Series는 이름표가 붙은 1차원 데이터다.

### Series 연산

**1. 정의**  
Series끼리 더하면 같은 인덱스끼리 값이 계산된다.

**2. 왜 필요한가?**  
날짜별 매출, 종목별 가격처럼 이름 기준으로 데이터를 맞춰 계산할 때 유용하다.

**3. 예시**

```python
stock0 = pd.Series([10, 20, 30], index=['naver', 'skt', 'LG'])
stock1 = pd.Series([30, 20, 10], index=['LG', 'naver', 'skt'])
merge = stock0 + stock1
print(merge)
```

```text
LG       60
naver    30
skt      30
dtype: int64
```

**4. 헷갈리기 쉬운 점**  
두 Series의 순서가 달라도 인덱스가 같으면 같은 항목끼리 계산된다. 순서가 아니라 이름표가 기준이다.

**5. 한 줄 정리**  
Series 연산은 위치보다 인덱스 이름을 우선한다.

## DataFrame

### 개념명

**1. 정의**  
DataFrame은 행과 열이 있는 2차원 표 데이터다. 데이터 분석에서 가장 자주 쓰는 형태다.

**2. 왜 필요한가?**  
실제 데이터는 고객명, 나이, 성별, 구매금액처럼 여러 열을 가진 표 형태가 많다. DataFrame은 이런 데이터를 다루기 쉽다.

**3. 예시**

```python
from pandas import Series, DataFrame

raw_data = {
    'col0': [1, 2, 3, 4],
    'col1': [10, 20, 30, 40],
    'col2': [100, 200, 300, 400]
}
data = DataFrame(raw_data)
print(data)
```

```text
   col0  col1  col2
0     1    10   100
1     2    20   200
2     3    30   300
3     4    40   400
```

**4. 헷갈리기 쉬운 점**  
DataFrame에서 행 이름은 index, 열 이름은 columns다. 둘을 구분하지 못하면 데이터를 선택할 때 실수가 잦다.

**5. 한 줄 정리**  
DataFrame은 데이터 분석의 기본 작업대다.

### loc와 iloc

**1. 정의**  
`loc`는 행과 열의 이름으로 데이터를 선택하고, `iloc`는 숫자 위치로 데이터를 선택한다.

**2. 왜 필요한가?**  
원하는 행과 열만 골라서 분석하거나 모델 입력으로 사용할 때 필요하다.

**3. 예시**

```python
data = {'A': [1, 2, 3], 'B': [4, 5, 6], 'C': [7, 8, 9]}
df = pd.DataFrame(data, index=['a', 'b', 'c'])

print(df.loc['a':'b', ['A', 'C']])
```

```text
   A  C
a  1  7
b  2  8
```

```python
print(df.iloc[0:2, [0, 2]])
```

```text
   A  C
a  1  7
b  2  8
```

**4. 헷갈리기 쉬운 점**  
`loc['a':'b']`는 끝값 `b`가 포함된다. 반면 `iloc[0:2]`는 끝 위치 2가 포함되지 않는다.

**5. 한 줄 정리**  
이름으로 고르면 loc, 순서 번호로 고르면 iloc다.

### drop과 sort_values

**1. 정의**  
`drop()`은 행 또는 열을 삭제하고, `sort_values()`는 특정 열을 기준으로 정렬한다.

**2. 왜 필요한가?**  
분석에 필요 없는 열을 제거하거나, 값의 크기 순서대로 데이터를 확인할 때 사용한다.

**3. 예시**

```python
data = {'A': [1, 2, 3], 'B': [4, 5, 6], 'C': [7, 8, 9]}
df = pd.DataFrame(data, index=['a', 'b', 'c'])

df_dropped = df.drop(labels='A', axis=1)
print(df_dropped)
```

```text
   B  C
a  4  7
b  5  8
c  6  9
```

```python
data = {'A': [3, 2, 1], 'B': [4, 5, 6]}
df = pd.DataFrame(data)
sorted_df = df.sort_values(by='A')
print(sorted_df)
```

```text
   A  B
2  1  4
1  2  5
0  3  6
```

**4. 헷갈리기 쉬운 점**  
`axis=0`은 행 방향, `axis=1`은 열 방향이다. 열을 삭제하려면 `axis=1`을 지정해야 한다.

**5. 한 줄 정리**  
drop은 표에서 필요 없는 줄이나 칸을 지우고, sort_values는 표를 기준 열에 맞춰 정렬한다.

## 코드로 보기 — 모델링용 데이터 선택

```python
cols = ['survived', 'pclass', 'sex', 'age', 'sibsp', 'parch', 'fare']
df_model = titanic[cols].copy()

print("Selected columns:")
print(df_model.head())
print()
print("Missing values before cleaning:")
print(df_model.isnull().sum())
```

**코드목적**  
타이타닉 데이터에서 모델 학습에 사용할 열만 선택한다.

**해석**  
전체 데이터에서 목표변수 `survived`와 입력변수로 쓸 `pclass`, `sex`, `age`, `sibsp`, `parch`, `fare`만 남긴다. 이후 결측치와 문자 데이터를 처리할 준비를 한다.

**실무 연결**  
모델링 전에 “어떤 컬럼을 쓸 것인가?”를 정하는 특성 선택 단계가 필요하다. Pandas는 이 과정을 가장 직관적으로 처리할 수 있게 해준다.

## 직접 해보기

1. `DataFrame`을 직접 만들고 특정 열 2개만 선택해보자.
2. `loc`와 `iloc`로 같은 결과를 만들어보자.
3. 결측치 개수를 확인하는 `isnull().sum()`을 사용해보자.

## 헷갈리기 쉬운 포인트

| A | B | 차이 |
| --- | --- | --- |
| Series | DataFrame | 1차원 데이터 vs 2차원 표 데이터 |
| index | columns | 행 이름 vs 열 이름 |
| loc | iloc | 이름 기반 선택 vs 위치 기반 선택 |
| axis=0 | axis=1 | 행 방향 vs 열 방향 |
| 원본 수정 | 복사본 수정 | `.copy()`를 쓰면 원본 영향을 줄일 수 있음 |

## 연결되는 개념

- 이전 글: [NumPy로 배열 데이터 다루기](02-numpy-array-basics.md)
- 다음 글: [Matplotlib과 Seaborn으로 시각화하기](04-visualization-matplotlib-seaborn.md)
- 다음 글: [타이타닉 생존 예측과 로지스틱 회귀](05-titanic-logistic-regression.md)
- 더 찾아볼 키워드: missing values, feature engineering, groupby, merge, one-hot encoding

## 셀프 체크

- [ ] Series와 DataFrame의 차이를 설명할 수 있다.
- [ ] DataFrame에서 특정 열을 선택할 수 있다.
- [ ] loc와 iloc의 차이를 설명할 수 있다.
- [ ] drop에서 axis의 의미를 설명할 수 있다.
- [ ] 머신러닝 전처리에서 Pandas가 왜 필요한지 말할 수 있다.

**복습 질문 및 답변**

Q. `loc`와 `iloc`의 가장 큰 차이는?  
A. `loc`는 이름 기준, `iloc`는 숫자 위치 기준이다.

Q. 열을 삭제하려면 `drop()`에서 어떤 axis를 써야 하는가?  
A. `axis=1`을 사용한다.

Q. Series끼리 더할 때 순서가 다르면 어떻게 되는가?  
A. 같은 인덱스끼리 맞춰서 계산된다.

## 한 줄 정리

> Pandas는 표 형태의 데이터를 모델링 가능한 형태로 정리하는 가장 중요한 전처리 도구다.
