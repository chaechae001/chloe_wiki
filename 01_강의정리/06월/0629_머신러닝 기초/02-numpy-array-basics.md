# NumPy로 배열 데이터 다루기
> 머신러닝 모델은 결국 숫자 배열을 입력으로 받는다. NumPy는 그 숫자 배열을 빠르고 일관되게 다루기 위한 기본 도구다.

`NumPy` · `array` · `arange` · `zeros` · `ones` · `reshape` · `slicing` · `concatenate` · `element-wise`

## 핵심요약

- NumPy 배열은 같은 자료형의 데이터를 순서대로 저장하는 자료 구조다.
- 파이썬 리스트와 달리 원소별 연산을 자연스럽게 처리할 수 있다.
- `np.array()`는 리스트를 배열로 변환한다.
- `np.arange()`, `np.zeros()`, `np.ones()`, `np.random`으로 다양한 배열을 만들 수 있다.
- `.shape`, `.size`, `.dtype`으로 배열의 구조와 자료형을 확인한다.
- `reshape()`는 원소 개수를 유지하면서 배열 형태를 바꾼다.
- 슬라이싱은 행과 열을 원하는 범위로 잘라내는 방법이다.
- 배열 연산에서 `*`는 원소별 곱이고, 행렬 곱은 `np.dot()` 또는 `@`를 사용한다.

## NumPy 배열

### 배열 생성

**1. 정의**  
배열은 같은 자료형의 데이터를 순서대로 저장하는 구조다. NumPy 배열은 다차원 데이터를 효율적으로 다룰 수 있게 해준다.

**2. 왜 필요한가?**  
머신러닝 모델은 많은 숫자 데이터를 빠르게 계산해야 한다. 리스트보다 NumPy 배열이 벡터 연산과 행렬 연산에 적합하다.

**3. 예시**

```python
import numpy as np

# 기본 1차원 배열 선언
a = np.array([0, 1, 2, 3])
print("Integer array:", a)

# dtype으로 float 타입 명시
b = np.array([0, 1, 2, 3], dtype=float)
print("Float array:", b)

# 중첩 리스트로 2차원 배열(행렬) 선언
c = np.array([[1, 2, 3], [4, 5, 6]])
print("2D array (matrix):")
print(c)
```

실행 결과는 다음과 같다.

```text
Integer array: [0 1 2 3]
Float array: [0. 1. 2. 3.]
2D array (matrix):
[[1 2 3]
 [4 5 6]]
```

**4. 헷갈리기 쉬운 점**  
파이썬 리스트에서 `[1, 2, 3] * 2`는 `[1, 2, 3, 1, 2, 3]`처럼 반복된다. NumPy 배열에서는 각 원소에 대해 계산이 일어난다.

**5. 한 줄 정리**  
NumPy 배열은 머신러닝 계산을 위한 숫자 데이터의 기본 그릇이다.

### 배열 생성 함수

**1. 정의**  
`arange`, `zeros`, `ones`, `random`은 반복되는 배열 생성 작업을 쉽게 해주는 함수다.

**2. 왜 필요한가?**  
분석이나 모델링에서는 일정 범위의 숫자, 0으로 초기화된 배열, 난수 배열을 자주 만든다.

**3. 예시**

```python
# arange 인자 1개: 0~9 (10개)
a = np.arange(10)
print("arange(10):", a)

# arange 인자 2개: 1~3 (4 미포함)
b = np.arange(1, 4)
print("arange(1, 4):", b)

# arange 인자 3개: 1~8, step=2 → 1,3,5,7
c = np.arange(1, 8, 2)
print("arange(1, 8, 2):", c)
```

```text
arange(10): [0 1 2 3 4 5 6 7 8 9]
arange(1, 4): [1 2 3]
arange(1, 8, 2): [1 3 5 7]
```

```python
zero_arr1 = np.zeros(5)
print("zeros(5) - float:", zero_arr1)

zero_arr2 = np.zeros(5, dtype=int)
print("zeros(5, dtype=int):", zero_arr2)

one_arr1 = np.ones(5)
print("ones(5) - float:", one_arr1)
```

**4. 헷갈리기 쉬운 점**  
`arange(start, stop, step)`에서 `stop`은 포함되지 않는다. `np.arange(1, 4)`의 결과는 `[1, 2, 3]`이다.

**5. 한 줄 정리**  
배열 생성 함수는 분석에 필요한 숫자 틀을 빠르게 만들어준다.

### 배열 정보 확인과 형태 변경

**1. 정의**  
`.shape`는 배열의 행과 열 크기, `.size`는 전체 원소 개수, `.dtype`은 자료형을 알려준다. `reshape()`는 배열의 모양을 바꾼다.

**2. 왜 필요한가?**  
머신러닝에서 입력 데이터의 shape가 맞지 않으면 모델 학습이 되지 않는다. 데이터가 몇 행 몇 열인지 확인하는 습관이 중요하다.

**3. 예시**

```python
c = np.array([[1, 2, 3], [4, 5, 6]])

print("Shape (rows, cols):", c.shape)
print("Size (total elements):", c.size)
print("Data type:", c.dtype)
print("Element at [1, 2]:", c[1, 2])
```

```text
Shape (rows, cols): (2, 3)
Size (total elements): 6
Data type: int64
Element at [1, 2]: 6
```

```python
a = np.array([1, 2, 3, 4])
a_reshaped = np.reshape(a, (2, 2))
print(a_reshaped)
```

```text
[[1 2]
 [3 4]]
```

**4. 헷갈리기 쉬운 점**  
`reshape()`는 원소 개수를 바꾸지 않는다. 원소가 4개인 배열은 `(2, 2)`로 바꿀 수 있지만 `(3, 2)`로는 바꿀 수 없다.

**5. 한 줄 정리**  
shape 확인은 데이터가 모델에 들어갈 준비가 되었는지 확인하는 첫 단계다.

### 슬라이싱과 연결

**1. 정의**  
슬라이싱은 배열에서 원하는 행과 열만 잘라내는 방법이다. 배열 연결은 여러 배열을 행 방향 또는 열 방향으로 붙이는 작업이다.

**2. 왜 필요한가?**  
특정 열만 모델 입력으로 쓰거나, 여러 데이터 조각을 합칠 때 필요하다.

**3. 예시**

```python
a0 = np.array([[1, 2, 3], [4, 5, 6]])

print("First column a0[:,0]:", a0[:, 0])
print("Last column a0[:,-1]:", a0[:, -1])
print("Second row a0[1,:]:", a0[1, :])
```

```text
First column a0[:,0]: [1 4]
Last column a0[:,-1]: [3 6]
Second row a0[1,:]: [4 5 6]
```

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
b = np.array([[7, 8, 9], [10, 11, 12]])

print(np.r_[a, b])
print(np.c_[a, b])
```

**4. 헷갈리기 쉬운 점**  
`axis=0`은 행 방향으로 아래에 붙이는 느낌이고, `axis=1`은 열 방향으로 옆에 붙이는 느낌이다.

**5. 한 줄 정리**  
슬라이싱은 필요한 부분만 꺼내는 가위, 연결은 배열을 붙이는 테이프다.

## 코드로 보기 — 배열 연산

```python
a = np.array([[1, 2, 3], [3, 2, 5]])
b = np.array([[-1, 3, 5], [1, 4, 2]])

print("a + b:")
print(a + b)

print("a - b:")
print(a - b)

print("a * b:")
print(a * b)

print("a / b:")
print(a / b)
```

**코드목적**  
두 배열의 같은 위치에 있는 원소끼리 계산한다.

**해석**  
`a + b`는 행렬 전체를 한 번에 더하지만, 내부적으로는 같은 위치의 숫자끼리 더한다. `a * b`도 행렬 곱이 아니라 원소별 곱이다.

**실무 연결**  
머신러닝에서는 특성 값 전체에 같은 계산을 적용하는 경우가 많다. NumPy의 원소별 연산은 반복문 없이 한 번에 계산하는 방식이다.

## 직접 해보기

1. `np.arange(1, 11)`로 1부터 10까지 배열을 만들고 `(2, 5)` 형태로 바꿔보자.
2. 2차원 배열에서 마지막 열만 추출해보자.
3. 같은 shape의 배열 두 개를 만들고 원소별 곱과 행렬 곱의 차이를 비교해보자.

## 헷갈리기 쉬운 포인트

| A | B | 차이 |
| --- | --- | --- |
| Python list | NumPy array | 리스트는 범용 자료구조, 배열은 수치 계산에 최적화 |
| `*` | `np.dot()` 또는 `@` | 원소별 곱 vs 행렬 곱 |
| `.shape` | `.size` | 구조 정보 vs 전체 원소 개수 |
| `np.r_` | `np.c_` | 행 방향 연결 vs 열 방향 연결 |
| `loc`/`iloc` | NumPy slicing | Pandas용 선택 방식 vs 배열 인덱싱 방식 |

## 연결되는 개념

- 이전 글: [인공지능과 머신러닝의 큰 그림](01-ai-machine-learning-basics.md)
- 다음 글: [Pandas로 표 데이터 다루기](03-pandas-dataframe-basics.md)
- 더 찾아볼 키워드: vectorization, broadcasting, matrix multiplication, ndarray

## 셀프 체크

- [ ] `np.array()`로 1차원과 2차원 배열을 만들 수 있다.
- [ ] `.shape`, `.size`, `.dtype`의 의미를 설명할 수 있다.
- [ ] `reshape()`에서 원소 개수가 유지되어야 한다는 점을 이해했다.
- [ ] `array[:, 0]`이 어떤 값을 가져오는지 설명할 수 있다.
- [ ] `*`와 행렬 곱의 차이를 설명할 수 있다.

**복습 질문 및 답변**

Q. `np.arange(1, 4)`의 결과가 `[1, 2, 3]`인 이유는?  
A. stop 값인 4는 포함되지 않기 때문이다.

Q. `(2, 3)` 배열의 `.size`는 얼마인가?  
A. 2×3이므로 6이다.

Q. 원소 12개 배열을 `reshape((2, -1))` 하면 열 개수는 어떻게 정해지는가?  
A. 전체 원소 12개를 2행으로 나누므로 열은 6개가 된다.

## 한 줄 정리

> NumPy는 머신러닝 입력 데이터의 기본 형태인 숫자 배열을 만들고, 자르고, 붙이고, 계산하는 도구다.
