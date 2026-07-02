# 데이터 변환·정제 — 타입 바꾸고 결측치 다루기

> 숫자처럼 보이는데 더해지지 않는 컬럼, 군데군데 비어 있는 값(NaN). 원본 데이터는 대체로 지저분합니다. 분석이 가능하도록 데이터를 "다듬는" 단계입니다.

`astype` `to_numeric` `to_datetime` `apply` `map` `fillna` `dropna` `sort_values`

## 핵심요약

- 데이터는 항상 원하는 타입으로 들어오지 않습니다. 숫자가 문자열(object)로 저장돼 연산이 안 되는 경우가 흔합니다.
- 타입 변환은 `astype()`(원하는 타입으로), `to_numeric()`(숫자로), `to_datetime()`(날짜·시간으로)을 씁니다.
- 날짜 타입이 되면 `dt.year`, `dt.month`, `dt.dayofweek`로 연·월·요일을 뽑을 수 있습니다.
- 컬럼 전체를 규칙대로 바꿀 땐 `map()`(딕셔너리 등), 함수를 적용할 땐 `apply()`를 씁니다. `lambda`로 함수를 한 줄로 쓸 수 있습니다.
- Pandas 메서드는 원본을 바로 바꾸지 않고 **결과를 반환**합니다. 원본을 바꾸려면 다시 대입(덮어쓰기)해야 합니다.
- 정렬은 `sort_values()`, 인덱스 정리는 `reset_index()`, 삭제는 `drop()`, 이름 변경은 `rename()`.
- 결측치는 `isnull()`로 찾고, `fillna()`로 채우거나 `dropna()`로 지웁니다.

---

## 데이터 타입 변환

### astype / to_numeric

**1. 정의**
컬럼의 자료형을 원하는 타입으로 바꿉니다. `astype(int)`처럼 직접 지정하거나, 숫자로 바꿀 때는 `to_numeric()`을 씁니다.

**2. 왜 필요한가?**
숫자가 텍스트(object)로 저장돼 있으면 덧셈·평균 같은 연산이 안 됩니다. 분석 전에 타입부터 맞춰야 합니다.

**3. 예시**

```python
# 방법 1: astype — 원하는 타입을 직접 지정
df["어른"] = df["어른"].astype(int)

# 방법 2: to_numeric — 숫자 타입으로 변환
df["유료합계"] = pd.to_numeric(df["유료합계"])

# 여러 컬럼을 한꺼번에 (반복문 활용)
columns = ['유료합계', '어른', '청소년', '어린이', '외국인', '단체', '무료합계', '총계']
for i in columns:
    df[i] = pd.to_numeric(df[i])
```

```python
# 문자열을 잘라 앞부분만 쓰고 싶을 때는 문자(str)로 다뤄야 한다
df_wel['unit_code'].astype(str).str[:2].head()  # 앞 2글자만
```

**4. 헷갈리기 쉬운 점**
`category` 타입은 새 값을 바로 넣을 수 없어 에러가 납니다. 이럴 땐 `astype('object')`로 바꾼 뒤 다루면 간단합니다.

**5. 한 줄 정리**
`astype`/`to_numeric`은 "연산이 되도록 타입을 맞추는" 작업입니다.

### to_datetime / dt

**1. 정의**
날짜처럼 생긴 문자열을 진짜 **시간 타입**으로 바꿉니다. 변환 후에는 `dt` 접근자로 연·월·일·요일을 꺼낼 수 있습니다.

**2. 왜 필요한가?**
"2016-01-01"이 문자열이면 월별·요일별 집계를 할 수 없습니다. 시간 타입으로 바꾸면 날짜 계산과 분해가 자유로워집니다.

**3. 예시**

```python
df["날짜"] = pd.to_datetime(df["날짜"])  # 문자열 → 시간 타입

df['연']  = df['날짜'].dt.year
df['월']  = df['날짜'].dt.month
df['일']  = df['날짜'].dt.day
df['요일'] = df['날짜'].dt.dayofweek  # 요일이 숫자로 (월~일 = 0~6)
```

**4. 헷갈리기 쉬운 점**
`dt.dayofweek`는 요일을 **숫자(0=월 … 6=일)** 로 줍니다. 글자로 바꾸려면 아래 `map`을 활용합니다.

**5. 한 줄 정리**
`to_datetime`은 "문자열 날짜를 계산 가능한 시간으로" 바꿉니다.

> 📅 비유: 문자열 "2016-01-01"은 **벽에 적힌 글씨**, 시간 타입은 **달력에 꽂힌 핀**입니다. 핀이라야 "이날이 무슨 요일?"을 물을 수 있습니다.

---

## 값을 바꾸는 도구 — map, apply, lambda

### map — 컬럼 전체를 규칙대로 치환

```python
# 숫자 요일을 글자로
week = {0:'월', 1:'화', 2:'수', 3:'목', 4:'금', 5:'토', 6:'일'}
df['요일'] = df['요일'].map(week)
```

딕셔너리의 Key→Value 규칙을 따라 컬럼 값을 한 번에 바꿉니다.

### apply — 컬럼에 함수를 적용

```python
def weather(e):
    if e == '눈' or e == '비':
        return '눈/비'
    else:
        return e

df['날씨'] = df['날씨'].apply(weather)  # '눈'과 '비'를 '눈/비'로 합침
```

`apply`는 `map`과 달리 복수의 컬럼에도 쓸 수 있고, `axis=0`이면 열 단위, `axis=1`이면 행 단위로 적용합니다(생략 시 0).

### lambda — 함수를 한 줄로

```python
# 위 weather 함수를 한 줄로
df['날씨'] = df['날씨'].apply(lambda e: '눈/비' if e == '눈' or e == '비' else e)
```

`lambda`는 콜론(`:`) 왼쪽이 입력, 오른쪽이 반환값입니다. 짧은 함수를 따로 `def` 없이 즉석에서 쓸 때 편합니다.

> ⚠️ **중요한 규칙**: Pandas 메서드는 대부분 원본을 바로 바꾸지 않고 **변환된 결과를 돌려줄 뿐**입니다. 원본을 진짜 바꾸려면 `df['x'] = df['x'].astype(int)`처럼 **다시 대입(덮어쓰기)** 해야 합니다.

---

## 파생 컬럼 만들기와 삭제

데이터프레임의 각 열은 Series라서, 컬럼끼리 사칙연산해 새 컬럼을 만들 수 있습니다.

```python
tips['인당결제금액'] = tips['total_bill'] / tips['size']  # 새 컬럼 생성

# 필요 없어지면 drop으로 삭제 (다시 대입해야 반영됨)
tips = tips.drop(columns=['인당결제금액'])
```

---

## 정렬·정리·이름 변경

```python
# 정렬: ascending=False면 내림차순 (생략 시 오름차순)
df_srv.sort_values('cpu_usage', ascending=False).head(8)
df_srv.sort_values(['location', 'check_date']).head(8)  # 여러 기준

# 인덱스 초기화: 정렬 후 뒤섞인 인덱스를 0부터 다시
df = df.reset_index(drop=True)

# 행/열 삭제: axis=0이면 행, axis=1이면 열
df = df.drop(["유료합계", "무료합계"], axis=1)

# 컬럼 이름 바꾸기
df = df.rename(columns={"총계": "총입장객수"})
```

---

## 결측치 다루기 — isnull, fillna, dropna

**1. 정의**
비어 있는 값(NaN)을 찾고(`isnull`/`isna`), 채우거나(`fillna`), 지우는(`dropna`) 작업입니다.

**2. 왜 필요한가?**
결측치가 있으면 평균·합계 같은 계산이 어긋나거나 에러가 납니다. 데이터를 신뢰하려면 결측을 먼저 처리해야 합니다.

**3. 예시**

```python
# 각 컬럼별 결측치 개수 (데이터가 커서 눈으로 찾기 어려우니 sum으로 집계)
df.isnull().sum()

import seaborn as sns
titanic = sns.load_dataset('titanic')
titanic.isna().sum()   # age 177, deck 688 ... 처럼 컬럼별 결측 개수
```

```python
# 채우기(fillna)
# 문자형 → 최빈값(mode), 수치형 → 평균/중앙값(mean/median)이 흔함
df = titanic.copy()
df['deck'] = df['deck'].astype('object')           # category → object
df['deck'] = df['deck'].fillna(df['deck'].mode()[0])  # 최빈값 'C'로 채움
df['age']  = df['age'].fillna(df['age'].median())     # 중앙값으로 채움
```

```python
# 삭제(dropna): 특정 컬럼에 결측 있는 행 제거 + 인덱스 초기화
df = df.dropna(subset=["청소년"], ignore_index=True)
```

**4. 헷갈리기 쉬운 점**
결측치를 **무조건 행 삭제(dropna)** 하면 데이터가 크게 줄어 손해입니다. 보통은 문자형은 최빈값, 수치형은 평균·중앙값으로 채우는 편을 먼저 고려합니다.

**5. 한 줄 정리**
결측치는 "찾고(isnull) → 채우거나(fillna) → 정 안 되면 지운다(dropna)"의 순서로 다룹니다.

---

## 코드로 보기 — 결측치를 평균으로 채우는 흐름

```python
df['청소년'] = df['청소년'].fillna(int(df['청소년'].mean()))
```

**코드목적**
'청소년' 컬럼의 빈 값을, 그 컬럼의 평균값(정수)으로 채워 넣습니다.

**해석**
안쪽부터 읽습니다. ① `df['청소년'].mean()`으로 평균을 구하고 → ② `int(...)`로 정수로 바꾼 뒤 → ③ `fillna(...)`로 결측 자리에 채우고 → ④ 다시 `df['청소년']`에 대입해 원본에 반영합니다.

**실무 연결**
설문 미응답, 센서 누락 같은 빈 값을 합리적인 대푯값으로 메워, 이후 집계·모델 학습이 끊기지 않게 합니다. 채우는 방식(평균/중앙값/최빈값)을 무엇으로 고르느냐가 분석 품질을 좌우합니다.

---

## 직접 해보기

1. `titanic` 데이터에서 `isna().sum()`으로 결측 컬럼을 찾고, 결측이 가장 많은 컬럼이 무엇인지 확인해 보세요.
2. 수치 컬럼 하나를 골라 결측치를 중앙값으로 채워 보세요.
3. 한 컬럼에 `map`으로 값 치환을, 같은 변환을 `apply(lambda ...)`로도 만들어 결과를 비교해 보세요.

---

## 헷갈리기 쉬운 포인트

- **반환 vs 원본 변경**: 메서드는 결과를 "반환"만 함. 원본을 바꾸려면 다시 대입(덮어쓰기).
- **map vs apply**: `map`은 한 컬럼에 규칙(딕셔너리) 치환, `apply`는 함수 적용(복수 컬럼 가능).
- **fillna vs dropna**: 채우면 데이터 보존, 지우면 데이터 감소. 보통 채우기를 먼저 고려.
- **astype(int) vs to_numeric**: 둘 다 숫자화에 쓰지만, 깨끗한 숫자면 `astype`, 변환이 까다로우면 `to_numeric`.

---

## 연결되는 개념

- 이전 글: 원하는 행만 골라내기 → [③ 조건 필터링](03-boolean-indexing-filtering.md)
- 다음 글: 다듬은 데이터를 그룹별로 묶고 합치기 → [⑤ 데이터 요약·병합](05-grouping-and-merging-data.md)
- 더 찾아볼 키워드: `replace`, `str` 접근자, `mode`, `interpolate`, `inplace`

---

## 셀프 체크

- [ ] 숫자가 문자열로 들어왔을 때 타입을 바꿀 수 있다.
- [ ] `to_datetime` 후 `dt`로 연·월·요일을 뽑을 수 있다.
- [ ] `map`과 `apply`의 차이를 설명할 수 있다.
- [ ] 메서드 결과를 원본에 반영하려면 다시 대입해야 함을 안다.
- [ ] `isnull`/`fillna`/`dropna`로 결측치를 처리할 수 있다.

**복습 질문 및 답변**

- **기본**: `df['x'].astype(int)`만 실행하면 원본이 바뀌나요?
  → 아닙니다. 변환된 결과만 반환됩니다. `df['x'] = df['x'].astype(int)`처럼 다시 대입해야 원본이 바뀝니다.
- **이해 확인**: `dt.dayofweek`의 결과는 무엇인가요?
  → 요일을 숫자(0=월요일 … 6=일요일)로 줍니다. 글자로 바꾸려면 `map`으로 딕셔너리를 적용합니다.
- **응용**: 문자형 컬럼과 수치형 컬럼의 결측치는 보통 어떻게 채우나요?
  → 문자형은 최빈값(`mode`), 수치형은 평균(`mean`)이나 중앙값(`median`)으로 채우는 경우가 많습니다.

---

## 한 줄 정리

> 변환·정제는 "타입을 맞추고(astype·to_datetime), 값을 바꾸고(map·apply), 결측을 다루는(fillna·dropna)" 다듬기 단계이며, 원본 반영은 다시 대입해야 합니다.
