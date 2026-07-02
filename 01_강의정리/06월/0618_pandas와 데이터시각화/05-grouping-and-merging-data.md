# 데이터 요약·병합 — groupby·concat·merge

> "지역별 평균 만족도", "요일별 매출 합계"처럼 그룹으로 묶어 요약하는 일은 분석의 꽃입니다. 그리고 흩어진 표 두 장을 합치면 더 풍부한 분석이 됩니다.

`groupby` `agg` `unstack` `concat` `merge` `Split-Apply-Combine`

## 핵심요약

- `groupby`는 데이터를 기준 컬럼으로 **나누고(Split) → 집계하고(Apply) → 합치는(Combine)** 과정입니다.
- 기본 문법은 `df.groupby('기준컬럼')['집계컬럼'].집계함수()` 입니다.
- 집계함수에는 `sum`, `count`, `size`, `mean`, `min`, `max`, `nunique` 등이 있습니다.
- 여러 통계를 한 번에 보려면 `agg(['mean', 'max', ...])`, 다중 인덱스를 표로 펼치려면 `unstack()`.
- 전체 통계 요약은 `describe()`, 개별 통계는 `mean()`·`min()`·`max()`·`median()`.
- 표를 위아래/좌우로 이어 붙일 땐 `concat()`, 특정 컬럼(key)을 기준으로 합칠 땐 `merge()`.
- `merge`의 `how`(inner/outer/left/right)는 "어느 쪽 데이터를 남길지"를 정합니다.

---

## groupby — 묶어서 요약하기

**1. 정의**
특정 컬럼 값에 따라 데이터를 그룹으로 나눈 뒤, 각 그룹에 집계 함수를 적용하고 결과를 합칩니다.

**2. 왜 필요한가?**
"전체 평균"보다 "그룹별 평균"이 훨씬 많은 인사이트를 줍니다. 요일별·지역별·카테고리별로 쪼개 보면 데이터의 차이가 드러납니다.

**3. 예시**

```python
# size별 total_bill 합계
tips.groupby('size')['total_bill'].sum()

# location별 log_id 개수
df_srv.groupby("location")['log_id'].count()
```

**4. 헷갈리기 쉬운 점**
그룹 기준 컬럼은 **반복되는 범주형**이 적절합니다(성별, 요일, 등급). `total_bill`처럼 고유값이 너무 많은 연속형 수치를 기준으로 삼으면 그룹이 의미를 잃습니다.

**5. 한 줄 정리**
`groupby`는 "기준별로 쪼개 요약하는" 도구입니다.

> 🍪 비유: 쿠키를 색깔별로 **나누고(Split)** → 색깔별 개수를 **세고(Apply)** → 결과표로 **합치는(Combine)** 것입니다.

### Split → Apply → Combine

```mermaid
flowchart LR
    A[전체 데이터] --> B[Split<br/>기준 컬럼으로<br/>그룹 나누기]
    B --> C[Apply<br/>그룹마다<br/>집계 함수 적용]
    C --> D[Combine<br/>결과를 하나로<br/>합치기]
    style A fill:#ede9fe,stroke:#7c3aed
    style D fill:#f5f3ff,stroke:#a78bfa
```

### 자주 쓰는 집계함수

| 함수 | 의미 |
| --- | --- |
| `sum()` | 합계 |
| `count()` | 개수 (결측 제외) |
| `size()` | 개수 (결측 포함) |
| `mean()` / `median()` | 평균 / 중앙값 |
| `min()` / `max()` | 최솟값 / 최댓값 |
| `std()` / `var()` | 표준편차 / 분산 |
| `nunique()` | 고유값 개수 |

```python
# 여러 기준으로 그룹화
tips.groupby(['day', 'sex'])['size'].count()   # 요일·성별 조합별 개수

df_srv.groupby('issue_category')['cpu_usage'].mean()  # 카테고리별 평균
```

> 💡 Pandas와 SQL은 사고방식이 같습니다.
> `df.groupby('category')['amount'].sum()` ≈ `SELECT category, SUM(amount) FROM t GROUP BY category;`

---

## agg — 여러 통계를 한 번에

집계값을 여러 개 동시에 보고 싶을 때 `agg()`에 리스트로 넘깁니다.

```python
df_srv.groupby('check_type')['fix_duration_hours'].agg(['mean', 'max'])
# 소수점 정리는 .round(2)
df_srv.groupby('check_type')['fix_duration_hours'].agg(['mean', 'max']).round(2)

df_sup.groupby('category')['requested_quantity'].agg(['sum', 'mean'])
```

집계 결과가 두 개 이상 컬럼이 되면 결과는 DataFrame이 됩니다.

## unstack — 다중 인덱스를 표로 펼치기

두 컬럼으로 그룹화하면 인덱스가 두 겹(다중 인덱스)이 되어 보기 불편합니다. `unstack()`으로 한 기준을 컬럼 방향으로 펼치면 피벗 테이블처럼 정리됩니다.

```python
result = df_wel.groupby(['region', 'facility_type'])['satisfaction_score'].mean().unstack()
result.loc["강원권", "독서실"]  # 펼친 표에서 특정 조합 값 조회
```

---

## 전체 통계 요약 — describe / mean

```python
df.mean()              # 모든 수치 컬럼의 평균
df["어른"].mean()       # 특정 컬럼의 평균
df.describe()          # count·mean·std·min·25/50/75%·max를 한 표로
```

`min()`, `max()`, `median()`도 같은 문법으로 씁니다.

---

## 표 합치기 — concat

**1. 정의**
공통 구조를 가진 데이터프레임을 위아래(또는 좌우)로 이어 붙입니다.

**2. 왜 필요한가?**
기간별로 나뉜 표(1월~3월, 4월)나 분산된 데이터를 하나로 모아 일관성 있게 분석하기 위해서입니다.

**3. 예시**

```python
df_concat = pd.concat([df, df2], axis=0, join='inner', ignore_index=True)
```

- `axis=0`: 위아래로(세로) 합침 (1이면 좌우)
- `join='inner'`: 두 표 모두에 있는 컬럼만 남김 (`outer`면 합집합, 기본값 outer)
- `ignore_index=True`: 기존 인덱스를 무시하고 0부터 새로 매김 (기본값 False)

**4. 헷갈리기 쉬운 점**
`concat`은 "단순히 이어 붙이기"입니다. 특정 컬럼(key)을 맞춰 가로로 연결하려면 `concat`이 아니라 `merge`를 씁니다.

**5. 한 줄 정리**
`concat`은 "구조가 같은 표를 이어 붙이는" 작업입니다.

> 📚 비유: `concat`은 같은 양식의 **공책 두 권을 위아래로 이어 붙이는** 것입니다.

---

## 표 합치기 — merge

**1. 정의**
특정 컬럼(key)을 기준으로 두 표를 가로로 연결합니다. SQL의 JOIN과 같습니다.

**2. 왜 필요한가?**
"입장객 데이터"에 "미세먼지 데이터"를 날짜 기준으로 붙이면, 날씨와 입장객을 함께 분석할 수 있습니다. 흩어진 정보를 하나의 표로 모읍니다.

**3. 예시**

```python
pd.merge(df, mm, on="날짜", how="left")
```

- `on`: 두 표를 맞출 기준 컬럼(key)
- `how`: 어느 쪽 행을 남길지 정하는 방식

| how | 의미 |
| --- | --- |
| `inner` | 양쪽에 모두 있는 key만 (교집합) |
| `outer` | 양쪽의 모든 key (합집합) |
| `left` | 왼쪽 표의 모든 행 기준 |
| `right` | 오른쪽 표의 모든 행 기준 |

**4. 헷갈리기 쉬운 점**
`how` 선택이 결과를 크게 바꿉니다. 한쪽에만 있는 key는 반대편 값이 `NaN`으로 채워집니다. "어느 표를 기준으로 남길지"를 먼저 정하세요.

**5. 한 줄 정리**
`merge`는 "공통 key로 두 표를 짝지어 가로로 잇는" 작업입니다.

> 🔑 비유: `merge`는 학번(key)을 맞춰 **성적표와 출석부를 한 줄씩 짝짓는** 것입니다.

### how를 그림으로

같은 두 표를 `on='월'`로 합칠 때, `how`에 따라 결과가 달라집니다.

| how | 남는 행 | 빈 칸 처리 |
| --- | --- | --- |
| inner | 두 표에 공통인 월만 | 빈 칸 없음 |
| outer | 양쪽의 모든 월 | 없는 쪽은 NaN |
| left | 왼쪽 표의 모든 월 | 오른쪽에 없으면 NaN |
| right | 오른쪽 표의 모든 월 | 왼쪽에 없으면 NaN |

```python
inner_df = pd.merge(df1, df2, on='월', how='inner')  # 공통만
outer_df = pd.merge(df1, df2, on='월', how='outer')  # 전부
left_df  = pd.merge(df1, df2, on='월', how='left')   # 왼쪽 기준
right_df = pd.merge(df1, df2, on='월', how='right')  # 오른쪽 기준
```

---

## 코드로 보기 — 입장객에 미세먼지 데이터 붙이기

```python
df_merge = pd.merge(df, mm, on="날짜", how="left")
```

**코드목적**
입장객 데이터(df)에, 같은 날짜의 미세먼지 데이터(mm)를 가로로 붙입니다.

**해석**
`on="날짜"`로 날짜를 맞추고, `how="left"`이므로 **왼쪽(입장객) 데이터가 있는 날짜만** 남깁니다. 분석의 핵심은 미세먼지가 아니라 입장객이므로, 입장객 기록이 없는 날(예: 휴장 기간)의 미세먼지는 의미가 없어 버리는 선택입니다.

**실무 연결**
서로 다른 출처의 데이터를 공통 key(날짜·ID 등)로 결합해 분석 범위를 넓히는, 실무에서 매우 자주 쓰는 작업입니다. `how`를 무엇으로 두느냐가 "어떤 행을 분석 대상으로 삼을지"를 결정합니다.

---

## 직접 해보기

1. `tips`에서 요일별 `total_bill` 평균을 `groupby`로 구해 보세요.
2. 한 그룹화 결과에 `agg(['mean', 'min', 'max'])`를 적용해 여러 통계를 한 번에 확인해 보세요.
3. 작은 두 표를 만들어 `merge`의 `inner`/`outer`/`left`/`right` 결과가 어떻게 달라지는지 비교해 보세요.

---

## 헷갈리기 쉬운 포인트

- **count vs size**: `count`는 결측 제외 개수, `size`는 결측 포함 전체 개수.
- **concat vs merge**: 단순 이어 붙이기는 `concat`, key로 짝지어 연결은 `merge`.
- **inner vs outer**: 교집합(공통만)은 `inner`, 합집합(전부)은 `outer`.
- **groupby 기준 선택**: 범주형은 적절, 고유값 많은 연속형 수치는 부적절.

---

## 연결되는 개념

- 이전 글: 분석 가능한 형태로 데이터 다듬기 → [④ 데이터 변환·정제](04-transforming-and-cleaning-data.md)
- 처음으로: 전체 흐름 다시 보기 → [① Pandas 시작하기](01-pandas-basics-and-inspecting-data.md)
- 더 찾아볼 키워드: `pivot_table`, `transform`, `join`, `crosstab`

---

## 셀프 체크

- [ ] groupby의 Split-Apply-Combine 흐름을 설명할 수 있다.
- [ ] `agg`로 여러 통계를 동시에 낼 수 있다.
- [ ] `unstack`이 왜 표를 보기 좋게 만드는지 안다.
- [ ] `concat`과 `merge`를 상황에 맞게 구분해 쓸 수 있다.
- [ ] `merge`의 `how` 4가지 차이를 설명할 수 있다.

**복습 질문 및 답변**

- **기본**: `df.groupby('day')['size'].count()`는 무엇을 구하나요?
  → 요일(day)별로 묶어, 각 요일의 size 개수를 셉니다.
- **이해 확인**: `concat`과 `merge`는 언제 각각 쓰나요?
  → 구조가 같은 표를 단순히 이어 붙일 땐 `concat`, 공통 key로 짝지어 가로로 연결할 땐 `merge`입니다.
- **응용**: 입장객 데이터에 날씨 데이터를 붙이되 입장객이 있는 날만 남기려면 `how`를 무엇으로?
  → 입장객(왼쪽) 데이터를 기준으로 남기는 `how='left'`를 씁니다.

---

## 한 줄 정리

> 요약은 `groupby`로 기준별 통계를 내는 것이고, 병합은 `concat`(이어 붙이기)·`merge`(key로 연결)로 흩어진 표를 하나로 모으는 것입니다.
