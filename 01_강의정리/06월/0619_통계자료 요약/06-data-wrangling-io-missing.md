# 데이터 가공 실무 — 파일 입출력과 결측치 처리

> 분석은 깨끗한 데이터에서만 매끄럽습니다. 그런데 현실의 데이터에는 빈칸(결측치)이 숨어 있고, 결과를 파일로 주고받는 일도 끊임없이 생깁니다. 분석가의 하루 중 절반은 사실 이 "가공"에 쓰입니다.

`파일입출력` `tocsv` `readexcel` `결측치` `isna` `fillna` `category타입` `파생변수`

## 핵심요약

- 가공한 데이터는 **CSV/Excel 파일로 저장(`to_csv`/`to_excel`)**하고 다시 **불러올 수 있어야(`read_csv`/`read_excel`)** 한다.
- 저장할 때 `index=False`를 빼먹으면, 다시 읽을 때 `Unnamed: 0` 같은 군더더기 컬럼이 생긴다.
- 결측치 처리는 "**어디에 얼마나 있는지 확인 → 처리 전략 선택**" 순서로 진행한다.
- 결측치는 `isna()`/`isnull()`/`notnull()`로 찾고, `sum()`·`any()`로 집계한다.
- 결측치를 채우는 `fillna()`는 컬럼의 **데이터 타입(dtype)**에 따라 다르게 동작한다. 특히 `category` 타입은 주의가 필요하다.
- 원본을 지키려면 처리 전에 `copy()`로 복사본을 만드는 습관이 좋다.

---

## 개념별 정리

### 파일 내보내기 — `to_csv` / `to_excel`

**1. 정의**
pandas로 가공한 데이터프레임을 CSV 또는 Excel 파일로 저장하는 기능입니다.

**2. 왜 필요한가?**
분석 결과를 동료에게 전달하거나, 다음 작업에서 다시 불러오려면 파일로 남겨야 합니다. pandas는 CSV·Excel뿐 아니라 JSON·HTML·XML·SQL 등 다양한 형식을 읽고 쓸 수 있습니다(공식 문서의 IO tools 참고).

**3. 예시**
```python
# 가공 결과를 복사본으로 보관 (원본 영향 방지)
result = df.copy()

# 저장 경로 = 폴더 경로 + 파일명 (문자열 결합)
data_path = "output/"                       # 폴더 경로 끝에 / 필요
result.to_csv(data_path + "result.csv", index=False)
result.to_excel(data_path + "result.xlsx", sheet_name="tips", index=False)
```

**4. 헷갈리기 쉬운 점**
- 폴더 경로 문자열 끝의 `/`를 빠뜨리면 파일명이 폴더명에 그대로 붙어 엉뚱한 위치에 저장됩니다.
- `index=False`를 지정하지 않으면 데이터프레임의 인덱스가 파일에 함께 저장되고, 나중에 읽을 때 `Unnamed: 0` 컬럼으로 들어옵니다.
- 저장해도 화면에 출력이 없는 것이 정상입니다. 파일 목록을 새로고침해 실제 생성됐는지 확인하세요.

**5. 한 줄 정리**
저장은 `to_csv`/`to_excel`, 군더더기 인덱스는 `index=False`로 막는다.

---

### 파일 불러오기 — `read_csv` / `read_excel`

**1. 정의**
저장된 파일을 다시 데이터프레임으로 읽어 들이는 기능입니다.

**2. 왜 필요한가?**
어제 만든 결과, 동료가 보낸 파일, 시스템에서 내려받은 데이터를 분석에 쓰려면 불러오기가 첫 관문입니다.

**3. 예시**
```python
import pandas as pd

data_path = "output/"
df = pd.read_csv(data_path + "result.csv")
df.head()

# Excel은 시트 이름을 지정해서 읽는다
df_excel = pd.read_excel(data_path + "result.xlsx", sheet_name="tips")
df_excel.head()
```

**4. 헷갈리기 쉬운 점**
- 읽었더니 `Unnamed: 0` 컬럼이 보인다면 → 저장 당시 `index=False`를 안 넣어 인덱스가 함께 저장된 흔적입니다.
- Excel은 파일 형식·환경에 따라 별도 엔진(`openpyxl`, `xlrd` 등)이 필요할 수 있습니다. 특히 매크로 포함 파일(`.xlsm`) 등 레거시 형식은 기본 설정으로 안 열릴 수 있어, "엔진 확인"이 핵심 포인트입니다.

**5. 한 줄 정리**
불러오기는 `read_csv`/`read_excel`, Excel은 `sheet_name`과 엔진을 확인한다.

> 비유: 파일 저장/불러오기는 택배 보내기/받기. 주소(경로)를 정확히 쓰고, 포장 규격(엔진)이 맞아야 받는 쪽에서 열린다.

---

### 결측치 탐색 — `isna` / `notnull` / `any`

**1. 정의**
- **결측치(Missing Value)**: 값이 비어 있는(NaN) 칸.
- `isna()`/`isnull()`: 각 칸이 결측인지 True/False로 표시(둘은 같은 역할).
- `notnull()`: 결측이 아닌 칸을 True로 표시.

**2. 왜 필요한가?**
결측치가 있으면 연산·통계 계산·모델 학습에서 오류가 나거나 결과가 누락됩니다. 그래서 "어디에 얼마나 있는지" 먼저 파악해야 합니다.

**3. 예시**
```python
df.isna()              # 칸별 결측 여부 (True/False)
df.isna().sum()        # 컬럼별 결측치 개수 (True를 1로 더함)

# 결측치가 하나라도 있는 컬럼명만 추출
df.columns[df.isnull().any()]

# 특정 컬럼의 결측 행 / 비결측 행
df[df["age"].isnull()]    # age가 비어 있는 행
df[df["age"].notnull()]   # age가 채워진 행
df[~df["age"].isnull()]   # notnull 대신 ~(부정) 연산자도 가능
```
공개된 승객 데이터(타이타닉)에서는 `age`에 177개, `deck`에 688개처럼 컬럼마다 결측치 개수가 다릅니다.

**4. 헷갈리기 쉬운 점**
- `isna().sum()`처럼 메서드를 이어 붙일 때는 쉼표가 아니라 **점(`.`)**으로 연결합니다.
- `sum()`이 결측치 개수를 세는 이유는, True를 1로 계산하기 때문입니다.
- `any()`는 "컬럼에 결측이 하나라도 있으면 True"를 돌려줍니다.

**5. 한 줄 정리**
결측은 `isna()`로 찾고, `sum()`으로 세고, `any()`로 컬럼 단위로 묻는다.

---

### 결측치 채우기 — `fillna`와 타입(dtype) 이슈

**1. 정의**
`fillna()`는 결측치를 특정 값으로 채우는 메서드입니다. 숫자형은 평균·중앙값으로, 범주형은 최빈값으로 채우는 경우가 많습니다.

**2. 왜 필요한가?**
결측이 있으면 분석이 막히므로, 삭제하거나 적절한 값으로 대체합니다. 단, "무엇으로 채울지"는 정답이 하나가 아니라 비교·검증으로 정합니다(전체 평균, 그룹별 평균, 예측 기반 추정 등).

**3. 예시**
```python
df_copy = df.copy()                 # 원본 보존

# 숫자형: 평균 또는 중앙값으로
df_copy["age"] = df_copy["age"].fillna(df_copy["age"].mean())

# 범주형: 최빈값으로 (value_counts로 검증 후)
top = df_copy["embarked"].mode()[0]
df_copy["embarked"] = df_copy["embarked"].fillna(top)
```

**4. 헷갈리기 쉬운 점 — `category` 타입의 함정**
타이타닉의 `deck` 컬럼은 `category` 타입입니다. `category`는 **허용된 범주 목록**을 미리 갖고 있어서, 목록에 없는 새 문자열로 바로 채우려 하면 에러가 납니다.
```python
df_copy["deck"].fillna("abc")       # ❌ category에 없는 값 → 에러 가능

# 해결 1: object(문자열) 타입으로 바꾼 뒤 채우기
df_copy["deck"] = df_copy["deck"].astype("object").fillna("abc")

# 해결 2: 카테고리를 먼저 추가한 뒤 채우기
# df_copy["deck"] = df_copy["deck"].cat.add_categories(["abc"]).fillna("abc")
```
핵심은 **에러 메시지를 보고 타입을 점검한 뒤 `astype`으로 변환**하면 풀린다는 점입니다. 같은 `fillna()`라도 dtype에 따라 동작이 달라지므로 타입 확인이 중요합니다.

**5. 한 줄 정리**
채우기 전 dtype을 확인하고, `category`는 타입 변환(`astype`) 또는 범주 추가로 푼다.

---

### 한 걸음 더 — 파생변수와 문자열 처리

**1. 정의**
- **파생변수**: 기존 컬럼을 조합해 새 컬럼을 만드는 것(예: 인당 결제금액 = 청구금액 ÷ 인원).
- **구간화(`pd.cut`)**: 연속형 값을 구간으로 나눠 Low/Mid/High 같은 범주로 바꾸는 것.
- **문자열 접근자(`.str`)**: 시리즈의 문자열에 문자열 메서드를 적용하는 통로.

**2. 왜 필요한가?**
원본 컬럼만으로는 부족할 때가 많습니다. 비율·구간·단어 포함 여부 같은 새 정보를 만들어야 분석이 풍부해집니다.

**3. 예시**
```python
# 파생변수: 인당 결제금액
df["per_person"] = df["total_bill"] / df["size"]

# 컬럼 삭제 (여러 개 동시 가능)
df = df.drop(columns=["per_person"])

# 연속형 구간화
df["bill_level"] = pd.cut(df["total_bill"],
                          bins=[0, 15, 30, 100],
                          labels=["Low", "Mid", "High"])

# 문자열 처리: 시리즈엔 .str로 접근해야 함
df["day"].str.upper()                      # 대문자
df["day"].str.contains("Sun", na=False).sum()  # 'Sun' 포함 개수 (SQL의 LIKE와 유사)
df["day"].str.len()                        # 문자열 길이
```

**4. 헷갈리기 쉬운 점**
시리즈에 `.upper()`를 바로 쓰면 에러가 납니다. 시리즈는 문자열 객체가 아니므로 **`.str`을 거쳐** `df["day"].str.upper()`처럼 써야 합니다. 또 이미 삭제한 컬럼을 다시 `drop`하면 `KeyError`가 납니다.

**5. 한 줄 정리**
새 정보는 파생변수·`pd.cut`으로 만들고, 문자열은 반드시 `.str`로 접근한다.

---

## 코드로 보기 — 결측치 처리의 표준 흐름

"확인 → 추출 → 복사 → 채우기" 순서로, 결측치를 다루는 한 사이클입니다.

```python
import pandas as pd

# 0) 원본 보존
df_work = df.copy()

# 1) 전체 결측치 개수 확인
print(df_work.isna().sum())

# 2) 결측치가 있는 컬럼만 골라 보기
cols_with_na = df_work.columns[df_work.isnull().any()]
print("결측 컬럼:", list(cols_with_na))

# 3) 특정 컬럼의 결측 행 확인
print(df_work[df_work["age"].isnull()].shape)   # 결측 행 수

# 4) 채우기 — 숫자형은 중앙값, 타입 주의 컬럼은 변환 후
df_work["age"] = df_work["age"].fillna(df_work["age"].median())
df_work["deck"] = df_work["deck"].astype("object").fillna("Unknown")

# 5) 처리 결과 재확인
print(df_work[["age", "deck"]].isna().sum())
```

**코드목적**
결측치가 어디에 얼마나 있는지 진단하고, 안전하게(원본 복사) 적절한 값으로 채운 뒤, 제대로 처리됐는지 검증하는 전체 절차입니다.

**해석**
1단계 `isna().sum()`으로 컬럼별 결측 개수를 보고, 2단계로 결측이 있는 컬럼만 추립니다. 4단계에서 숫자형 `age`는 중앙값(이상값에 강건)으로 채우고, `category` 타입인 `deck`은 `astype("object")`로 바꾼 뒤 채웁니다. 마지막에 다시 `isna().sum()`을 찍어 0으로 줄었는지 확인하면, 작업이 의도대로 됐는지 알 수 있습니다.

**실무 연결**
조건 필터링, 그룹화 연산, 결측 데이터 처리는 데이터 분석의 가장 기본기이자 거의 모든 실무 과제의 출발점입니다. 결측치를 어떻게 채우느냐(전체 평균 vs 그룹별 평균 vs 예측값)에 따라 이후 통계와 모델 결과가 달라지므로, 한 가지 방법을 맹신하기보다 여러 방식을 비교하는 태도가 중요합니다.

---

## 직접 해보기

1. 데이터프레임을 `to_csv`로 저장할 때 `index=False`를 넣은 경우와 뺀 경우를, 다시 `read_csv`로 읽어 컬럼 차이를 확인해 보세요.
2. `df.isna().sum()`으로 결측이 가장 많은 컬럼을 찾고, 숫자형이면 중앙값으로 채워 보세요.
3. `category` 타입 컬럼에 새 문자열을 `fillna`로 바로 넣어 에러를 재현한 뒤, `astype("object")`로 해결해 보세요.

## 헷갈리기 쉬운 포인트

- **`isna()` vs `notnull()`**: 전자는 결측이 True, 후자는 비결측이 True. `~df["x"].isna()`는 `notnull()`과 같다.
- **`index=False` 있음 vs 없음**: 없으면 다시 읽을 때 `Unnamed: 0` 컬럼이 생긴다.
- **시리즈 `.upper()` vs `.str.upper()`**: 시리즈엔 문자열 메서드를 직접 못 쓴다. `.str`을 거쳐야 한다.
- **`fillna` on object vs category**: object는 아무 값이나, category는 등록된 범주만. 새 값은 타입 변환 후.

## 연결되는 개념

- 이전 글: [두 변수의 관계 요약하기](05-summarize-two-variables.md) — 깨끗이 정제한 데이터라야 상관·분포 분석이 신뢰할 만하다.
- 함께 보기: [숫자로 한 변수 요약하기](04-summarize-one-variable-numbers.md) — 결측치를 중앙값으로 채울 때 그 중앙값의 의미.
- 처음으로: [강의 한눈에 보기](../OVERVIEW.md) — 하루 전체 흐름 다시 보기.
- 더 찾아볼 키워드: `dropna`, `pd.cut의 bins/labels`, `str.contains(LIKE)`, `openpyxl/xlrd 엔진`, `groupby 평균으로 결측 채우기`

## 셀프 체크

- [ ] `to_csv`/`to_excel`로 저장하고 `read_csv`/`read_excel`로 불러올 수 있다.
- [ ] `index=False`가 필요한 이유를 안다.
- [ ] `isna()`·`any()`·`sum()`으로 결측치를 진단할 수 있다.
- [ ] `category` 타입에서 `fillna` 에러가 나는 이유와 해결법을 안다.
- [ ] 처리 전 `copy()`로 원본을 보존하는 습관이 있다.

**복습 질문 및 답변**

- (기본) 컬럼별 결측치 개수를 한 줄로 구하려면? → `df.isna().sum()`.
- (이해확인) 저장한 CSV를 다시 읽었더니 `Unnamed: 0` 컬럼이 생긴 이유는? → 저장 시 `index=False`를 지정하지 않아 인덱스가 파일에 함께 저장됐기 때문.
- (응용) `category` 타입 컬럼의 결측치를 등록되지 않은 새 문자열로 채우려면? → 바로 채우면 에러가 나므로 `astype("object")`로 타입을 바꾼 뒤 `fillna`하거나, `cat.add_categories`로 범주를 추가한 뒤 채운다.

## 한 줄 정리

> 분석의 절반은 가공이다. 결과는 `index=False`로 깔끔히 저장·불러오고, 결측치는 "확인 → 복사 → 타입 맞춰 채우기" 순서로 다룬다.
