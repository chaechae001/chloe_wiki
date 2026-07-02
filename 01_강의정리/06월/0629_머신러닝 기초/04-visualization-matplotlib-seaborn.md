# Matplotlib과 Seaborn으로 시각화하기
> 데이터 분석은 숫자를 계산하는 일만이 아니라, 패턴을 눈으로 확인하고 설득력 있게 전달하는 일이다.

`Matplotlib` · `Seaborn` · `plot` · `scatter` · `hist` · `bar` · `subplot` · `regplot` · `countplot` · `jointplot`

## 핵심요약

- Matplotlib은 Python의 기본 시각화 라이브러리다.
- `plt.plot()`은 선그래프, `plt.scatter()`는 산점도, `plt.hist()`는 히스토그램, `plt.bar()`는 막대그래프를 그린다.
- 제목, 축 이름, 범례를 추가하면 그래프 해석이 쉬워진다.
- `plt.subplots()`는 여러 그래프를 한 화면에 배치할 때 사용한다.
- Seaborn은 Matplotlib 기반의 통계 시각화 라이브러리다.
- `regplot()`은 산점도와 회귀선을 함께 보여준다.
- `countplot()`은 범주형 데이터의 빈도를 확인할 때 유용하다.
- `jointplot()`은 두 변수의 관계와 각 변수의 분포를 동시에 보여준다.

## Matplotlib 기본 그래프

### 선그래프, 산점도, 히스토그램, 막대그래프

**1. 정의**  
Matplotlib은 데이터를 그래프로 그리는 가장 기본적인 도구다. 데이터의 변화, 관계, 분포, 크기 비교를 표현할 수 있다.

**2. 왜 필요한가?**  
숫자 표만 보면 패턴을 놓치기 쉽다. 그래프는 이상치, 추세, 분포, 그룹 차이를 빠르게 확인하게 해준다.

**3. 예시**

```python
import matplotlib.pyplot as plt

plt.plot(["Seoul", "Paris", "Seattle"], [30, 25, 55])
plt.xlabel('City')
plt.ylabel('Response')
plt.title('Experiment Result')
plt.show()
```

```python
plt.scatter([1, 3, 2], [0, -1, 2])
plt.show()
```

```python
plt.hist([1, 3, 2, 2, 3, 3, 3, 1, 4, 8, 2, 6, 6], bins=2)
plt.show()
```

```python
plt.bar(["Seoul", "Paris", "Seattle"], [30, 25, 55])
plt.xlabel('City')
plt.ylabel('Response')
plt.title('Experiment Result')
plt.show()
```

**4. 헷갈리기 쉬운 점**  
그래프 종류는 데이터 질문에 맞춰 선택해야 한다. 시간 흐름은 선그래프, 두 숫자 변수의 관계는 산점도, 분포는 히스토그램, 범주별 크기 비교는 막대그래프가 적합하다.

**5. 한 줄 정리**  
Matplotlib은 데이터를 눈으로 확인하기 위한 기본 도화지다.

### 여러 그래프 배치

**1. 정의**  
`subplots()`는 한 화면에 여러 그래프를 배치할 때 사용한다.

**2. 왜 필요한가?**  
생존 여부별 그래프, 성별 그래프, 등급별 그래프처럼 여러 관점을 한 번에 비교할 수 있다.

**3. 예시**

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

sns.countplot(x='survived', data=titanic, ax=axes[0])
axes[0].set_title('Survival Count')

sns.countplot(x='pclass', hue='survived', data=titanic, ax=axes[1])
axes[1].set_title('Survival by Passenger Class')

plt.tight_layout()
plt.show()
```

**4. 헷갈리기 쉬운 점**  
`axes[0]`, `axes[1]`은 각각 첫 번째, 두 번째 그래프 영역이다. 그래프를 어디에 그릴지 `ax=`로 지정해야 한다.

**5. 한 줄 정리**  
subplots는 여러 그래프를 한 장의 리포트처럼 배치해준다.

## Seaborn 통계 시각화

### regplot

**1. 정의**  
`regplot()`은 산점도와 회귀선을 함께 보여주는 함수다.

**2. 왜 필요한가?**  
두 수치형 변수 사이에 증가 또는 감소 관계가 있는지 빠르게 확인할 수 있다.

**3. 예시**

```python
import seaborn as sns
import matplotlib.pyplot as plt

df = sns.load_dataset('iris')

sns.regplot(x='sepal_length', y='petal_length', data=df)
plt.title('Sepal Length vs Petal Length')
plt.show()
```

**4. 헷갈리기 쉬운 점**  
회귀선이 있다고 해서 반드시 인과관계가 있다는 뜻은 아니다. 그래프는 관계를 보여줄 뿐, 원인과 결과를 증명하지는 않는다.

**5. 한 줄 정리**  
regplot은 두 숫자 변수의 관계를 산점도와 추세선으로 함께 보여준다.

### countplot과 jointplot

**1. 정의**  
`countplot()`은 범주형 데이터의 개수를 세어 막대그래프로 보여준다. `jointplot()`은 두 변수의 산점도와 각 변수의 분포를 함께 보여준다.

**2. 왜 필요한가?**  
범주별 빈도와 두 변수의 관계는 EDA에서 가장 먼저 확인하는 내용이다.

**3. 예시**

```python
titanic = sns.load_dataset('titanic')

sns.countplot(x='class', data=titanic)
plt.title('Passenger Count by Class')
plt.show()
```

```python
sns.countplot(x='class', hue='survived', data=titanic)
plt.title('Passenger Count by Class and Survival')
plt.legend(['Not Survived (0)', 'Survived (1)'])
plt.show()
```

```python
tips = sns.load_dataset('tips')
sns.jointplot(x='total_bill', y='tip', data=tips, kind='reg')
plt.show()
```

**4. 헷갈리기 쉬운 점**  
`countplot()`은 이미 집계된 값이 아니라 원본 데이터의 빈도를 세어준다. 이미 집계된 값을 막대로 그리고 싶다면 `barplot()` 또는 Matplotlib의 `bar()`를 고려한다.

**5. 한 줄 정리**  
countplot은 범주별 개수, jointplot은 두 변수의 관계와 분포를 동시에 보여준다.

## 코드로 보기 — 타이타닉 데이터 EDA

```python
numeric_cols = ['survived', 'pclass', 'age', 'sibsp', 'parch', 'fare']
corr_matrix = titanic[numeric_cols].corr()

plt.figure(figsize=(8, 6))
sns.heatmap(corr_matrix, annot=True, fmt='.2f', cmap='RdBu_r',
            center=0, linewidths=0.5)
plt.title('Correlation Matrix of Titanic Features')
plt.tight_layout()
plt.show()

print("Correlation with 'survived':")
print(corr_matrix['survived'].sort_values(ascending=False))
```

**코드목적**  
타이타닉 데이터의 수치형 변수들이 생존 여부와 어떤 상관관계를 갖는지 확인한다.

**해석**  
실행 결과에서 `fare`는 생존과 양의 상관관계, `pclass`는 생존과 음의 상관관계를 보였다. 즉 요금이 높을수록 생존과 함께 움직이는 경향이 있고, 등급 숫자가 커질수록 생존과 반대로 움직이는 경향이 있다.

```text
Correlation with 'survived':
survived    1.000000
fare        0.257307
parch       0.081629
sibsp      -0.035322
age        -0.077221
pclass     -0.338481
Name: survived, dtype: float64
```

**실무 연결**  
모델링 전에 시각화와 상관분석을 하면 어떤 변수가 예측에 도움 될지 감을 잡을 수 있다. 이는 서비스 기획에서 “어떤 데이터를 수집해야 하는가?”를 판단하는 근거가 된다.

## 직접 해보기

1. 타이타닉 데이터에서 성별별 생존 건수를 `countplot()`으로 그려보자.
2. `age`와 `fare`의 분포를 각각 히스토그램으로 확인해보자.
3. `jointplot()`의 `kind='scatter'`, `kind='reg'` 결과를 비교해보자.

## 헷갈리기 쉬운 포인트

| A | B | 차이 |
| --- | --- | --- |
| plot | scatter | 선으로 연결 vs 점으로 표시 |
| hist | bar | 분포 확인 vs 범주별 크기 비교 |
| Matplotlib | Seaborn | 기본 도화지 vs 통계 그래프 도구 |
| countplot | barplot | 원본 빈도 집계 vs 집계/평균값 표현 |
| 상관관계 | 인과관계 | 함께 움직임 vs 원인과 결과 |

## 연결되는 개념

- 이전 글: [Pandas로 표 데이터 다루기](03-pandas-dataframe-basics.md)
- 다음 글: [타이타닉 생존 예측과 로지스틱 회귀](05-titanic-logistic-regression.md)
- 더 찾아볼 키워드: EDA, correlation, heatmap, distribution, regression line

## 셀프 체크

- [ ] 선그래프, 산점도, 히스토그램, 막대그래프의 용도를 구분할 수 있다.
- [ ] `xlabel`, `ylabel`, `title`, `legend`의 역할을 설명할 수 있다.
- [ ] Seaborn의 `regplot`, `countplot`, `jointplot`을 구분할 수 있다.
- [ ] 상관관계가 인과관계를 의미하지 않는다는 점을 이해했다.
- [ ] EDA가 모델링 전에 필요한 이유를 설명할 수 있다.

**복습 질문 및 답변**

Q. 생존 여부별 승객 수를 확인하려면 어떤 그래프가 적합한가?  
A. 범주형 변수의 빈도를 확인하는 `countplot()`이 적합하다.

Q. 두 수치형 변수의 관계를 보고 싶다면?  
A. 산점도 또는 `regplot()`을 사용할 수 있다.

Q. 히트맵은 언제 유용한가?  
A. 여러 수치형 변수 간 상관관계를 한눈에 보고 싶을 때 유용하다.

## 한 줄 정리

> 시각화는 모델링 전에 데이터의 구조와 패턴을 눈으로 확인하고, 분석 결과를 설득력 있게 전달하는 과정이다.
