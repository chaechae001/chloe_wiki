# 조건 필터링(Boolean indexing) — 원하는 행만 골라내기

> "점수 80점 이상인 학생", "디스크 문제가 발생한 서버"처럼 조건에 맞는 행만 남기고 싶을 때가 분석의 90%입니다. Pandas는 이걸 어떻게 처리할까요?

`Boolean indexing` `조건식` `isin` `between` `reset_index` `copy`

## 핵심요약

- 조건 필터링은 각 행마다 조건이 참인지(True) 거짓인지(False) 판단해, **True인 행만 남기는** 방식입니다.
- `df['점수'] > 80` 같은 조건식은 데이터를 바로 줄이는 게 아니라, 행마다 True/False가 든 **Boolean Series**를 만듭니다.
- 조건이 여러 개면 각 조건을 괄호로 감싸고 `&`(and), `|`(or), `~`(not)으로 잇습니다.
- 여러 후보 값 중 하나면 `isin()`, 수치 구간이면 `between()`을 씁니다.
- 한 컬럼을 다른 컬럼과 비교하는 **컬럼끼리 비교**도 가능합니다.
- 필터링 결과는 원본 인덱스를 유지하므로, 따로 저장할 땐 `reset_index(drop=True)` + `.copy()`가 표준 패턴입니다.

---

## Boolean indexing의 원리

**1. 정의**
조건식의 결과(True/False)를 이용해, 참인 행만 골라내는 인덱싱입니다.

**2. 왜 필요한가?**
실제 분석은 "전체"가 아니라 "조건에 맞는 일부"를 보는 일이 대부분입니다. 특정 지역, 특정 기간, 특정 카테고리만 추려 내는 가장 기본적인 도구입니다.

**3. 예시**

```python
df_srv['issue_category'] == '디스크'
# → 행마다 True/False가 담긴 Series가 만들어진다 (아직 데이터가 줄지 않음)

df_srv.loc[df_srv['issue_category'] == '디스크', :]
# → True인 행만 남는다 (= 디스크 문제가 있는 행)
```

**4. 헷갈리기 쉬운 점**
값을 비교할 때는 대입 연산자 `=`이 아니라 **비교 연산자 `==`** 를 써야 합니다. `=`는 "넣어라", `==`는 "같은가?"입니다.

**5. 한 줄 정리**
조건식은 행마다 참/거짓을 매기고, Pandas는 참인 행만 남깁니다.

> 🚪 비유: 조건식은 입구의 **검표원**입니다. "디스크 문제 있음" 도장이 찍힌 행만 통과시킵니다.

---

## 핵심 구조: df.loc[행조건, 열선택]

필터링은 `loc` 안에서 "행 자리에 조건식, 열 자리에 원하는 컬럼"을 넣는 형태로 씁니다.

```python
df.loc[df['점수'] >= 80, '이름']
# 점수가 80 이상인 행만 남기고, 그중 '이름' 컬럼만 선택
```

```python
# 수치 비교 예시
tips.loc[tips['total_bill'] >= 50, :]  # 결제액 50 이상
df_srv.loc[df_srv['cpu_usage'] > 40, :].head()
```

---

## 복합 조건 — &, |, ~

조건이 여러 개일 때는 각 조건을 **괄호로 감싸고** 비트 연산자로 잇습니다.

| 의미 | 파이썬 | Pandas |
| --- | --- | --- |
| 그리고 | `and` | `&` |
| 또는 | `or` | `|` |
| 부정 | `not` | `~` |

```python
# total_bill 30 이상 '이면서' time이 Dinner
tips.loc[(tips['total_bill'] >= 30) & (tips['time'] == 'Dinner'), :]

# 두 조건이 모두 1인 행
df_srv.loc[(df_srv['fix_required'] == 1) & (df_srv['issue_detected'] == 1), :]
```

> ⚠️ 부등호는 한 번에 하나만 비교합니다. `(30 < A < 50)`은 안 되고, `(A > 30) & (A < 50)`처럼 둘로 나눠야 합니다. 괄호를 먼저 만들어 두고 조건을 채우면 실수가 줄어듭니다.

---

## isin — 여러 후보 값 중 하나

같은 컬럼에서 `==`를 여러 번 쓰는 대신, 후보 값을 리스트로 한 번에 검사합니다. SQL의 `IN`과 같습니다.

```python
# check_type이 '일간' 또는 '주간'인 행
df_srv.loc[df_srv['check_type'].isin(['일간', '주간']), :]

# ~ 를 붙이면 '아닌' 것 (NOT IN)
df_srv.loc[~df_srv['check_type'].isin(['일간', '주간']), :]
```

## between — 수치 구간

점수·온도·금액처럼 "어디부터 어디까지"를 찾을 때 씁니다.

```python
# mental_wellbeing_score가 6 이상 8 이하
df_wel.loc[df_wel['mental_wellbeing_score'].between(6, 8), :].head()

# & 로 직접 쓴 것과 같은 의미
df_wel.loc[(df_wel['mental_wellbeing_score'] >= 6) &
           (df_wel['mental_wellbeing_score'] <= 8), :].head()
```

> 📖 공식 문서를 직접 읽고 예제를 타이핑해 보는 습관이 중요합니다. Pandas 문서는 보통 **설명 → 반환값 → 예제** 순서로 구성돼 있어, 예제를 따라 치며 결과를 확인하면 빠르게 익힙니다.

---

## 컬럼끼리 비교

상수가 아니라 **다른 컬럼의 값**과 비교할 수도 있습니다.

```python
import seaborn as sns
iris = sns.load_dataset('iris')

# 두 컬럼을 행 단위로 비교 → True/False
iris['sepal_width'] >= iris['petal_length']

# 비교 결과를 새 컬럼으로 추가
iris2 = iris.loc[:, ['sepal_width', 'petal_length']].copy()
iris2['비교'] = iris2['sepal_width'] >= iris2['petal_length']
iris2.loc[iris2['비교'] == True]
```

```python
# 재고가 재주문 기준보다 적은(=재주문 필요한) 품목만
df_sup.loc[df_sup['stock_level'] < df_sup['reorder_threshold'],
           ['item_code', 'item_name', 'stock_level', 'reorder_threshold']]
```

---

## 표준 패턴 — reset_index(drop=True) + .copy()

조건 필터링을 하면 살아남은 행들이 **원래 인덱스 번호를 그대로 유지**합니다. 예를 들어 11,000개 중 2,700개가 남으면 인덱스가 0, 1, 2…가 아니라 띄엄띄엄(예: 0, 5, 12…) 남습니다. 분석용으로 따로 저장할 땐 인덱스를 0부터 다시 정리하고, 원본과 분리해 둡니다.

```python
df_sup2 = df_sup.loc[df_sup['category'] == '의료', :].reset_index(drop=True).copy()
```

- `reset_index(drop=True)`: 기존 인덱스를 버리고 0부터 다시 매김 (`drop=True`는 대괄호가 아니라 `reset_index(...)` 소괄호 안 옵션)
- `.copy()`: 원본과 분리된 복사본 생성

```python
# 외워 두면 좋은 한 줄 패턴
new_df = df.loc[조건식, 선택컬럼].reset_index(drop=True).copy()
```

> 참고로 어떤 값들이 있는지 종류만 빠르게 보려면 `unique()`를 씁니다(SQL의 `DISTINCT`와 비슷). 종류와 함께 개수까지 보려면 `value_counts()`입니다.

---

## 코드로 보기 — 빈도 최상위 값으로 필터링

```python
import seaborn as sns
tips = sns.load_dataset("tips")

# size 컬럼에서 빈도가 가장 적은 값을 찾아, 그 값만 추출
least_size = tips['size'].value_counts().index[-1]  # 가장 적게 등장한 size
tips.loc[tips['size'] == least_size, :]
```

**코드목적**
"가장 드문 그룹만 떼어 보기"처럼, 분포 확인(`value_counts`)과 필터링을 이어서 쓰는 패턴입니다.

**해석**
`value_counts()`는 빈도 내림차순으로 정렬되므로, `.index[0]`은 가장 많은 값, `.index[-1]`은 가장 적은 값입니다. 이 값을 조건에 넣어 해당 행만 추출합니다.

**실무 연결**
"가장 많이 팔린 상품군만", "응답이 가장 적은 지역만"처럼 분포의 양 끝을 떼어 깊게 보는 분석에서 자주 쓰입니다.

---

## 직접 해보기

1. `tips` 데이터에서 `smoker`가 `'No'`인 행만 추출하고 인덱스를 초기화해 보세요.
2. `size >= 3` 이면서 `tip >= 3`인 행을 복합 조건으로 추출해 보세요.
3. 한 수치 컬럼을 골라 `between()`으로 특정 구간을, 같은 결과를 `&`로도 만들어 비교해 보세요.

---

## 헷갈리기 쉬운 포인트

- **`=` vs `==`**: 대입은 `=`, 비교는 `==`. 필터링에는 `==`.
- **`and`/`or` vs `&`/`|`**: Pandas 조건 결합은 `&`, `|`(각 조건은 괄호 필수).
- **`isin` vs `between`**: 후보 값 목록이면 `isin`, 수치 구간이면 `between`.
- **`unique()` vs `value_counts()`**: 종류만 보면 `unique`, 종류+개수면 `value_counts`.

---

## 연결되는 개념

- 이전 글: `loc`/`iloc`의 기본 사용법 → [② loc와 iloc](02-loc-and-iloc.md)
- 다음 글: 골라낸 데이터를 다듬고 결측치를 다루기 → [④ 데이터 변환·정제](04-transforming-and-cleaning-data.md)
- 더 찾아볼 키워드: `query()`, `notnull`/`isnull`, `where`, `mask`

---

## 셀프 체크

- [ ] 조건식이 만드는 Boolean Series의 의미를 설명할 수 있다.
- [ ] 복합 조건을 `&`/`|`/`~`와 괄호로 작성할 수 있다.
- [ ] `isin`과 `between`을 상황에 맞게 쓸 수 있다.
- [ ] 컬럼끼리 비교해 조건을 만들 수 있다.
- [ ] `reset_index(drop=True) + .copy()` 표준 패턴을 안다.

**복습 질문 및 답변**

- **기본**: `df['점수'] > 80`은 무엇을 반환하나요?
  → 행마다 True/False가 담긴 Boolean Series입니다. 이걸 `loc`에 넣으면 True인 행만 남습니다.
- **이해 확인**: `(30 < A < 50)`이 안 되는 이유는?
  → 부등호는 한 번에 하나만 비교할 수 있어, `(A > 30) & (A < 50)`처럼 둘로 나눠야 합니다.
- **응용**: 필터링 후 인덱스가 띄엄띄엄 남는 이유와 해결법은?
  → 살아남은 행이 원래 인덱스를 유지하기 때문입니다. `reset_index(drop=True)`로 0부터 다시 정리합니다.

---

## 한 줄 정리

> 조건 필터링은 행마다 참/거짓을 매겨 참인 행만 남기는 작업이며, `reset_index(drop=True).copy()`로 깔끔히 저장합니다.
