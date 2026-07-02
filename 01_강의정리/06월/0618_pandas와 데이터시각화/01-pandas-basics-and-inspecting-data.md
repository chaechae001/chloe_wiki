# Pandas 시작하기 — 데이터프레임·시리즈와 데이터 살펴보기

> 표 한 장을 받았습니다. 행이 1만 개라면, 무엇부터 봐야 할까요? 눈으로 스크롤하지 않고 데이터의 "건강 상태"를 30초 만에 파악하는 방법이 있습니다.

`Pandas` `DataFrame` `Series` `read_csv` `info` `describe` `value_counts`

## 핵심요약

- Pandas는 **표(행·열) 형태의 데이터를 다루는 파이썬 라이브러리**로, "엑셀의 파이썬 버전"이라고 부를 만합니다.
- 핵심 자료구조는 두 개입니다. **DataFrame**(2차원 표), **Series**(열 하나, 1차원).
- 데이터프레임에서 컬럼 하나를 뽑으면 보통 **Series**, 여러 컬럼을 리스트로 뽑으면 **DataFrame** 이 됩니다.
- 파일을 불러올 때는 `read_csv()`(csv), `read_excel()`(xlsx)을 씁니다.
- 데이터를 처음 만나면 `head`/`tail`(맛보기), `info`(스키마), `describe`(통계), `shape`(규모)로 상태를 점검합니다.
- 특정 컬럼의 값 분포는 `value_counts()`로 한 번에 셀 수 있습니다.
- 큰 화면이 깨지지 않게, 시각화 전에는 한글 폰트 설정을 먼저 해 둡니다.

---

## DataFrame과 Series

### DataFrame (데이터프레임)

**1. 정의**
행(가로)과 열(세로)로 이루어진 2차원 표입니다. 우리가 엑셀에서 보는 그 표라고 생각하면 됩니다.

**2. 왜 필요한가?**
현실의 데이터 대부분은 표 형태입니다. 입장객 기록, 매출 장부, 설문 응답 모두 "행은 한 건, 열은 항목"으로 정리됩니다. 데이터프레임은 이 표를 코드로 자유롭게 자르고 계산하게 해 줍니다.

**3. 예시**

```python
import pandas as pd

df = pd.read_csv("data/server.csv")
df.head()
```

**4. 헷갈리기 쉬운 점**
표 전체는 DataFrame이지만, 거기서 컬럼 하나만 뽑으면 더 이상 표가 아니라 Series가 됩니다. 결과 형태가 헷갈리면 `type()`으로 확인하는 습관이 좋습니다.

**5. 한 줄 정리**
DataFrame은 "행과 열을 가진 표 한 장"입니다.

> 🫓 비유: 데이터프레임은 **엑셀 시트 한 장**, 각 열은 그 시트의 한 칼럼입니다.

### Series (시리즈)

**1. 정의**
열 하나로 이루어진 1차원 데이터입니다. 인덱스(번호·이름)와 값으로 구성됩니다.

**2. 왜 필요한가?**
"점수 컬럼의 평균", "지역 컬럼의 빈도"처럼 **한 열만 대상으로 연산할 때** 자연스럽게 Series를 다루게 됩니다.

**3. 예시**

```python
type(df_wel)            # pandas.core.frame.DataFrame
type(df_wel['region'])  # pandas.core.series.Series  ← 컬럼 하나면 Series
```

**4. 헷갈리기 쉬운 점**
`df['region']`(대괄호 1개)은 Series, `df[['region']]`(대괄호 2개)은 컬럼이 하나여도 DataFrame입니다. 대괄호 개수 하나로 형태가 바뀝니다.

**5. 한 줄 정리**
Series는 "표에서 떼어낸 컬럼 한 줄"입니다.

> 📏 비유: 데이터프레임이 시트라면, Series는 그 시트에서 **세로로 오려낸 한 칼럼**입니다.

---

## 데이터 불러오기와 살펴보기

### read_csv / read_excel

**1. 정의**
파일로 저장된 데이터를 데이터프레임으로 불러오는 함수입니다. csv는 `read_csv()`, 엑셀(xlsx)은 `read_excel()`을 씁니다.

**2. 왜 필요한가?**
분석은 "데이터를 메모리로 가져오는 것"에서 시작합니다. 파일 경로만 정확하면 한 줄로 표가 만들어집니다.

**3. 예시**

```python
import pandas as pd
df = pd.read_csv("data/server.csv")  # 파일을 불러와 df라는 변수에 표로 저장
```

**4. 헷갈리기 쉬운 점**
경로 오타, 슬래시(`/`) 누락, 파일명 대소문자 차이가 가장 흔한 에러 원인입니다(`FileNotFoundError`). 경로는 직접 타이핑하기보다 복사해서 붙여넣는 편이 안전합니다.

**5. 한 줄 정리**
`read_csv`는 "파일 → 표"로 바꾸는 첫 단추입니다.

### 데이터 점검 메서드 모음

데이터를 불러온 직후에는 아래 메서드들로 "이 데이터가 어떤 모양인지"를 빠르게 확인합니다.

| 메서드 | 무엇을 보나 |
| --- | --- |
| `df.shape` | (행 개수, 열 개수) — 데이터 규모 |
| `df.info()` | 컬럼 이름, 자료형(dtype), 결측치 개수 |
| `df.describe()` | 수치형 열의 평균·최소·최대·사분위수 |
| `df.head()` / `df.tail()` | 위/아래 일부 행 미리보기 |
| `df.columns.tolist()` | 컬럼 이름을 리스트로 |
| `df.dtypes` | 컬럼별 자료형 |
| `df.sample(n, random_state=)` | 무작위 표본 |

```python
df_srv.shape          # 예: (11000, 14)
df_srv.shape[0]       # 행 개수
df_srv.shape[1]       # 열 개수

df_sup.info()         # 스키마 한눈에: SQL의 DESC와 비슷
df_wel.describe()     # 수치형 요약 통계
df_wel.describe(include='object')  # 범주형(문자열) 컬럼 요약

len(df_srv), len(df_sup), len(df_wel)  # 각 표의 행 수 비교
```

> **자료형(dtype) 미리 알기**: `int`(정수), `float`(실수), `bool`(참/거짓), `datetime`(날짜·시간), `category`(범주), `object`(문자열·혼합). 날짜가 `object`로 들어와 있으면 나중에 변환이 필요하다는 신호입니다. → [④ 데이터 변환·정제](04-transforming-and-cleaning-data.md)

`sample()`은 실행할 때마다 다른 행이 나오는데, `random_state`를 고정하면 매번 같은 결과가 나옵니다.

```python
df_wel.sample(5)                  # 매번 달라짐
df_wel.sample(5, random_state=42) # 고정 → 재현 가능 (42는 그냥 정한 숫자)
```

### 특정 컬럼 추출

데이터프레임에서 컬럼이름으로 원하는 열을 뽑을 수 있습니다.

```python
df["어른"]                 # 컬럼 하나 → Series
df[["공휴일", "어른"]]     # 리스트로 묶으면 → DataFrame(여러 컬럼)
```

### value_counts — 값별 개수 세기

**1. 정의**
한 컬럼 안에서 각 값이 몇 번 등장하는지 세어, 빈도 높은 순으로 보여 줍니다.

**2. 왜 필요한가?**
요일·지역·카테고리처럼 반복되는 값의 분포를 빠르게 파악할 때 유용합니다. "데이터가 어디에 쏠려 있는지"를 한눈에 봅니다.

**3. 예시**

```python
df_wel['region'].value_counts()
# SELECT region, count(*) FROM df_wel GROUP BY region ORDER BY 2 DESC 와 같은 의미
```

**4. 헷갈리기 쉬운 점**
`unique()`는 "어떤 값들이 있는지"(종류)만, `value_counts()`는 "각 값이 몇 번 나오는지"(종류+개수)까지 알려 줍니다. → 자세한 비교는 [③ 조건 필터링](03-boolean-indexing-filtering.md)

**5. 한 줄 정리**
`value_counts()`는 "이 컬럼 값들이 어떻게 분포하는지" 보는 빠른 도구입니다.

---

## 코드로 보기 — 데이터를 처음 만났을 때의 점검 루틴

```python
import pandas as pd

df = pd.read_csv("data/server.csv")

df.shape            # ① 얼마나 큰가?
df.info()           # ② 어떤 컬럼이, 무슨 타입으로, 결측은 없는가?
df.head()           # ③ 실제 값은 어떻게 생겼나?
df.describe()       # ④ 숫자 컬럼의 분포는?
df['location'].value_counts()  # ⑤ 특정 컬럼은 어디에 쏠려 있나?
```

**코드목적**
처음 보는 데이터의 규모·구조·상태·분포를 순서대로 빠르게 파악하기 위한 루틴입니다.

**해석**
`shape`로 규모를, `info`로 결측·타입을, `head`로 실제 값을, `describe`로 숫자 분포를, `value_counts`로 특정 컬럼의 쏠림을 봅니다. 이 다섯 줄이면 "이 데이터로 무엇을 할 수 있는지" 감이 잡힙니다.

**실무 연결**
분석·머신러닝 프로젝트의 첫 단계인 EDA(탐색적 데이터 분석)가 바로 이 점검에서 출발합니다. 결측이 많은 컬럼, 타입이 잘못 들어온 컬럼을 여기서 미리 발견하면 뒤 작업이 훨씬 수월해집니다.

---

## 시각화 준비 — 한글 폰트 설정

그래프에 한글을 쓰면 글자가 □□□로 깨지는 경우가 많습니다. 리눅스 기반 환경(예: 클라우드 노트북)에서는 한글 폰트를 먼저 설치해 두면 이후 시각화에서 깨짐을 막을 수 있습니다.

```python
# 리눅스/클라우드 환경에서 한글 폰트 설치 (로컬 Windows는 생략 가능)
!sudo apt-get install -y -qq fonts-nanum
!fc-cache -fv
!rm ~/.cache/matplotlib -rf
# 설치 후 세션(런타임)을 다시 시작한 뒤 다음 코드를 실행
```

폰트를 설치하고 세션을 다시 시작했다면, 매번 설치 셀을 다시 실행할 필요는 없습니다. 이 설정은 본격적인 데이터 시각화(그래프 그리기)를 위한 사전 준비 단계입니다.

---

## 직접 해보기

1. 임의의 csv를 불러와 `shape`, `info()`, `describe()`를 차례로 출력하고, 결측치가 있는 컬럼이 무엇인지 찾아보세요.
2. 한 컬럼을 `df['컬럼']`과 `df[['컬럼']]`로 각각 뽑아 `type()`으로 결과 형태가 어떻게 다른지 확인해 보세요.
3. 범주형으로 보이는 컬럼 하나를 골라 `value_counts()`로 가장 많이/적게 등장한 값을 확인해 보세요.

---

## 헷갈리기 쉬운 포인트

- **DataFrame vs Series**: 표 전체는 DataFrame, 컬럼 하나는 Series. `df['x']`(Series) vs `df[['x']]`(DataFrame).
- **info() vs describe()**: `info`는 구조(타입·결측), `describe`는 숫자 분포. 목적이 다릅니다.
- **unique() vs value_counts()**: 종류만 보려면 `unique`, 종류+개수를 보려면 `value_counts`.

---

## 연결되는 개념

- 다음 글: 원하는 행·열을 콕 집어 뽑는 방법 → [② loc와 iloc](02-loc-and-iloc.md)
- 이어지는 글: 조건으로 거르기 → [③ 조건 필터링](03-boolean-indexing-filtering.md)
- 더 찾아볼 키워드: `EDA`, `read_excel`, `pd.set_option`(출력 옵션), `seaborn` 내장 데이터셋(`tips`, `iris`, `titanic`)

---

## 셀프 체크

- [ ] DataFrame과 Series의 차이를 한 문장으로 설명할 수 있다.
- [ ] `read_csv`로 파일을 불러와 변수에 저장할 수 있다.
- [ ] `info()`로 결측치와 자료형을 확인할 수 있다.
- [ ] 컬럼 하나(Series)와 여러 개(DataFrame)를 구분해 뽑을 수 있다.
- [ ] `value_counts()`로 값의 분포를 확인할 수 있다.

**복습 질문 및 답변**

- **기본**: 컬럼 하나를 뽑으면 어떤 자료구조가 되나요?
  → Series입니다(1차원). 여러 컬럼을 리스트로 뽑으면 DataFrame입니다.
- **이해 확인**: `info()`와 `describe()`는 무엇이 다른가요?
  → `info()`는 컬럼 이름·자료형·결측치 개수 같은 "구조"를 보여 주고, `describe()`는 수치형 컬럼의 평균·최소·최대 같은 "분포 통계"를 보여 줍니다.
- **응용**: 어떤 컬럼이 `object` 타입인데 내용은 숫자라면, 무엇을 의심해야 하나요?
  → 숫자가 문자열로 저장된 경우입니다. 그대로면 덧셈·평균 등 연산이 안 되므로 타입 변환이 필요합니다([④](04-transforming-and-cleaning-data.md)).

---

## 한 줄 정리

> Pandas의 출발점은 "데이터를 표(DataFrame)로 불러와, 규모·구조·분포를 빠르게 점검하는 것"입니다.
