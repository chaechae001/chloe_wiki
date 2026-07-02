# 데이터 결합으로 피처 테이블 만들기

> 현실의 데이터는 한 파일에 깔끔하게 모여 있지 않습니다. 주문 따로, 고객 따로, 결제 따로. 분석의 절반은 이렇게 **흩어진 표를 올바르게 합치는 일**입니다. pandas의 병합 도구 전체를 한 번에 정리합니다.
> 이 글은 수학 공식 자체보다, 벡터·행렬 형태의 모델 입력을 만들기 전 흩어진 데이터를 하나의 분석 테이블로 정리하는 실무 준비 단계에 가깝다.


`pandas` · `merge` · `join` · `concat` · `inner` · `outer` · `merge_asof` · `combine_first`

## 핵심요약

- `pd.concat()` 은 같은 구조의 데이터를 위아래(`axis=0`) 또는 좌우(`axis=1`)로 **단순히 이어붙인다**.
- `pd.merge()` 는 **공통 키(열)** 를 기준으로 SQL 스타일 조인을 한다.
- 조인 방식 네 가지: `inner`(교집합), `left`(왼쪽 전부), `right`(오른쪽 전부), `outer`(합집합).
- 매칭되지 않는 행은 `NaN`(결측)으로 채워진다.
- 컬럼명이 다르면 `left_on`/`right_on`, 키가 인덱스면 `DataFrame.join()` 을 쓴다.
- 시계열엔 `merge_ordered`(정렬 병합)·`merge_asof`(가장 가까운 값 매칭)가 있다.
- `combine_first`(결측 보완)·`compare`(버전 비교)는 데이터 정합성 점검에 쓴다.

> 실습 데이터는 주문·고객·제품·결제 등으로 구성된 공개 샘플 전자상거래 데이터셋(CSV)을 사용합니다. 테이블 구조(주문↔고객↔제품)는 실제 업무 데이터와 거의 같아 연습에 적합합니다.

```python
import pandas as pd
import numpy as np

# 예: 같은 폴더의 CSV들을 불러옵니다 (경로는 환경에 맞게 수정)
DATA = "./erd_sample"
orders       = pd.read_csv(f"{DATA}/orders.csv", parse_dates=["orderDate", "shippedDate"])
customers    = pd.read_csv(f"{DATA}/customers.csv")
orderdetails = pd.read_csv(f"{DATA}/orderdetails.csv")
products     = pd.read_csv(f"{DATA}/products.csv")
payments     = pd.read_csv(f"{DATA}/payments.csv", parse_dates=["paymentDate"])
```

---

## 개념별 정리

### concat — 단순 이어붙이기

**1. 정의**
같은 구조의 데이터를 행 방향(세로로 쌓기, `axis=0`) 또는 열 방향(가로로 붙이기, `axis=1`)으로 이어붙입니다.

**2. 왜 필요한가?**
분기별·지역별로 나뉜 동일 형식의 파일을 하나로 합칠 때 가장 간단합니다. "키로 매칭"이 아니라 "그냥 쌓기"가 목적일 때 씁니다.

**3. 예시**

```python
# 분기별로 나뉜 주문을 세로로 합치기
orders_q1 = orders[orders["orderDate"] < "2003-02-01"]
orders_q2 = orders[orders["orderDate"] >= "2003-02-01"]

result = pd.concat([orders_q1, orders_q2], ignore_index=True)
print(len(orders_q1), len(orders_q2), len(result))   # 5 5 10
```

**4. 헷갈리기 쉬운 점**
`ignore_index=True` 를 안 주면 원래 인덱스가 그대로 따라와 0,1,2,0,1,2 처럼 중복됩니다. `axis=1`(가로 붙이기)은 **인덱스를 기준**으로 정렬해 붙이므로, 인덱스가 안 맞으면 `NaN` 이 생깁니다.

**5. 한 줄 정리**
`concat` 은 같은 모양의 데이터를 위아래/좌우로 쌓는 도구다.
> 🍞 비유: 1월·2월·3월 장부를 그냥 아래로 이어 붙여 1분기 통합 장부를 만드는 것입니다.

---

### merge — 공통 키로 조인 (inner/left/right/outer)

**1. 정의**
두 표의 **공통 키(예: 고객번호)** 를 기준으로 옆으로 합칩니다. SQL의 JOIN과 같으며, `how` 옵션으로 어떤 행을 남길지 정합니다.

**2. 왜 필요한가?**
"주문 표"에는 고객번호만 있고 이름은 "고객 표"에 있습니다. 둘을 고객번호로 합쳐야 "누가 무엇을 주문했는지" 분석할 수 있습니다.

**3. 예시 — 네 가지 조인**

```python
# INNER: 양쪽에 모두 있는 고객번호만
inner = pd.merge(orders, customers[["customerNumber","customerName","country"]],
                 on="customerNumber", how="inner")

# LEFT: 왼쪽(orders) 전부 유지, 없는 고객 정보는 NaN
left = pd.merge(orders, customers[["customerNumber","customerName"]],
                on="customerNumber", how="left")

# RIGHT: 오른쪽(customers) 전부 유지 → 주문 없는 고객도 포함
right = pd.merge(orders, customers, on="customerNumber", how="right")

# OUTER: 양쪽 전부 유지
outer = pd.merge(orders, customers, on="customerNumber", how="outer")
```

같은 두 표라도 결과 행 수가 달라집니다. 예컨대 10건씩이라도 매칭되는 고객이 4명뿐이면 `inner` 결과는 4행, `left` 는 10행(고객 정보 6건은 `NaN`), `outer` 는 양쪽을 다 합쳐 16행이 됩니다.

**4. 헷갈리기 쉬운 점**
`inner` 는 매칭 안 되는 행을 버리므로 데이터가 줄어듭니다. 분석 중 행이 갑자기 줄면 십중팔구 의도치 않은 `inner` 조인 탓입니다. "왼쪽 데이터를 모두 지키고 싶다"면 `left` 가 안전합니다.

**5. 한 줄 정리**
`merge` 는 공통 키로 표를 잇되, `how` 가 "누구를 남길지"를 결정한다.

| `how` | 결과에 남는 행 | NaN 발생 |
|-------|---------------|---------|
| `inner` | 양쪽 다 키가 있는 행만 | 없음 |
| `left` | 왼쪽 전부 + 오른쪽 매칭 | 오른쪽 |
| `right` | 오른쪽 전부 + 왼쪽 매칭 | 왼쪽 |
| `outer` | 양쪽 전부 | 양쪽 |

> 🍞 비유: 학번으로 '수강 명단'과 '성적표'를 합치는 엑셀 VLOOKUP인데, 명단에만 있는 학생을 남길지(left) 양쪽 다 있는 학생만 남길지(inner)를 고르는 것입니다.

---

### 다중 키·체인 merge, 컬럼명이 다를 때

**1. 정의**
키가 두 개 이상이면 `on=["col1","col2"]`, 컬럼명이 서로 다르면 `left_on`/`right_on`, 세 개 이상 표는 `.merge()` 를 연달아 이어(체인) 붙입니다.

**2. 왜 필요한가?**
실제 데이터는 표 두 개로 끝나지 않습니다. 주문상세 → 제품 → 제품카테고리처럼 줄줄이 연결해야 "카테고리별 매출" 같은 분석이 됩니다.

**3. 예시**

```python
# 체인 merge: 주문상세 → 제품 → 제품라인
chain = (orderdetails
         .merge(products[["productCode","productName","productLine"]], on="productCode")
         .merge(productlines[["productLine","textDescription"]],       on="productLine"))

# 컬럼명이 다를 때: 직원의 상사 정보 붙이기(self-join)
emp_with_manager = pd.merge(
    employees,
    employees[["employeeNumber","lastName"]].rename(columns={
        "employeeNumber":"mgrNumber", "lastName":"mgrLastName"}),
    left_on="reportsTo", right_on="mgrNumber", how="left")
```

**4. 헷갈리기 쉬운 점**
같은 표를 자기 자신과 합치는 self-join에서는 컬럼명이 겹치므로 한쪽 이름을 바꿔(`rename`) 줘야 헷갈리지 않습니다. 체인 merge는 중간에 키가 빠지면 매칭이 끊기니 단계별로 행 수를 확인하세요.

**5. 한 줄 정리**
키가 여럿이거나 표가 여럿이어도 `on`·`left_on/right_on`·체인으로 모두 연결할 수 있다.
> 🍞 비유: 영수증 → 상품 → 상품분류로 꼬리에 꼬리를 무는 연결고리를 따라가는 것입니다.

---

### join — 인덱스 기반 조인

**1. 정의**
`DataFrame.join()` 은 인덱스를 기준으로 붙이는 편의 문법입니다(`merge(left_index=True, ...)` 의 간편 버전).

**2. 왜 필요한가?**
키가 이미 인덱스로 설정된 표끼리는 `join` 이 더 짧고 직관적입니다.

**3. 예시**

```python
prod_idx  = products.set_index("productCode")[["productName","productLine","buyPrice"]]
pline_idx = productlines.set_index("productLine")[["textDescription"]]

joined = prod_idx.join(pline_idx, on="productLine", how="left")
```

**4. 헷갈리기 쉬운 점**
`join` 은 기본이 인덱스 기준입니다. 일반 컬럼을 키로 쓰려면 `on=` 을 명시해야 합니다. 컬럼명이 겹치면 `lsuffix`/`rsuffix` 로 구분해야 오류가 안 납니다.

**5. 한 줄 정리**
`join` 은 "인덱스가 키일 때" 쓰는 merge의 단축키다.

---

### 시계열 병합 — merge_ordered·merge_asof

**1. 정의**
`merge_ordered` 는 병합 결과를 **정렬된 순서로** 유지하고 누락값을 앞 값으로 채울 수 있습니다(`fill_method='ffill'`). `merge_asof` 는 정확히 일치하는 키가 없을 때 **가장 가까운 이전/이후 값**으로 매칭합니다.

**2. 왜 필요한가?**
"주문일 직전의 가장 최근 결제", "특정 시각 직전의 주가"처럼 시간 축에서 근사 매칭이 필요한 경우가 많습니다. 일반 `merge` 로는 정확히 같은 시각만 매칭되어 대부분 누락됩니다.

**3. 예시**

```python
# 각 주문에 대해, 같은 고객의 '주문일 이전 가장 최근 결제' 매칭
ord_sorted = orders[["orderNumber","customerNumber","orderDate"]].sort_values("orderDate")
pay_sorted = payments[["customerNumber","paymentDate","amount"]].sort_values("paymentDate")

asof = pd.merge_asof(
    ord_sorted, pay_sorted,
    left_on="orderDate", right_on="paymentDate",
    by="customerNumber",        # 같은 고객 안에서만 매칭
    direction="backward")       # 주문일 '이전' 결제
```

**4. 헷갈리기 쉬운 점**
`merge_asof` 는 **양쪽 모두 키 기준으로 미리 정렬**되어 있어야 합니다. `direction` 은 `backward`(이전)·`forward`(이후)·`nearest`(가장 가까운 쪽) 중에서 목적에 맞게 고릅니다.

**5. 한 줄 정리**
시간 데이터엔 정렬을 지키는 `merge_ordered`, 가까운 값을 잇는 `merge_asof` 를 쓴다.
> 🍞 비유: "이 사건이 일어난 직전에 마지막으로 한 행동"을 찾아 짝지어 주는 것이 `merge_asof` 입니다.

---

### combine_first·compare — 보완과 비교

**1. 정의**
`combine_first` 는 A를 우선하되 A가 `NaN` 인 곳만 B 값으로 채웁니다. `compare` 는 두 DataFrame에서 **달라진 셀만** 골라 보여줍니다.

**2. 왜 필요한가?**
여러 소스에서 모은 데이터의 빈칸을 서로 메우거나(`combine_first`), 업데이트 전후에 무엇이 바뀌었는지 빠르게 확인할 때(`compare`) 씁니다.

**3. 예시**

```python
# 결측 보완: source_a 우선, a가 NaN인 곳만 source_b로 채움
filled = source_a.combine_first(source_b)

# 버전 비교: 변경된 셀만 표시
diff = orders_v1.compare(orders_v2, result_names=("이전","이후"))
print(f"변경된 행: {len(diff)}건")
```

**4. 헷갈리기 쉬운 점**
`combine_first` 는 "A 우선"이라는 방향이 있습니다. A에 값이 있으면 B와 다르더라도 A 값을 유지합니다. `compare` 는 기본적으로 **달라진 행만** 보여주며, 전체를 보려면 `keep_shape=True` 를 줍니다.

**5. 한 줄 정리**
`combine_first` 는 빈칸 메우기, `compare` 는 달라진 곳 찾기다.

---

## 코드로 보기 — 체인 merge로 카테고리별 매출 구하기

여러 표를 연결해 "제품 카테고리별 매출"이라는 실전 지표를 뽑아 봅니다.

```python
chain = (
    orderdetails
    .merge(products[["productCode","productName","productLine"]], on="productCode")
    .merge(productlines[["productLine","textDescription"]],       on="productLine")
)

sales_by_line = (
    chain
    .assign(revenue=lambda d: d["quantityOrdered"] * d["priceEach"])
    .groupby("productLine", as_index=False)["revenue"]
    .sum()
    .sort_values("revenue", ascending=False)
)
print(sales_by_line)
```

실행 결과(예시 데이터 기준):

```text
    productLine   revenue
0  Classic Cars  19525.55
2  Vintage Cars  10054.80
1   Motorcycles   3732.30
```

**코드목적**
주문상세(수량·단가)에 제품 정보와 카테고리 정보를 차례로 붙인 뒤, 카테고리별 매출(`수량 × 단가`)을 합산·정렬합니다.

**해석**
표 세 개를 연결하지 않으면 "주문상세"에는 카테고리가 없어 이런 집계가 불가능합니다. 체인 merge로 정보를 모은 덕분에 Classic Cars가 매출 1위라는 결론을 얻습니다. 분석의 핵심 정보는 대개 **여러 표에 흩어져** 있고, 병합이 그 정보를 한자리에 모읍니다.

**실무 연결**
매출 대시보드, 코호트 분석, 고객별 LTV 계산 등 거의 모든 분석이 "여러 테이블 조인 → 집계" 순서입니다. 어떤 조인을 쓰느냐에 따라 결과가 달라지므로, 병합은 분석 신뢰도의 첫 단추입니다.

---

## 직접 해보기

1. `orders` 와 `customers` 를 `customerNumber` 로 `inner`/`left` 조인했을 때 결과 행 수가 왜 달라지는지 설명해 보세요.
2. "주문이 한 건도 없는 고객"까지 결과에 포함하려면 어떤 `how` 를 써야 할까요?
3. 두 분기 주문 표를 세로로 합칠 때, 인덱스가 0,1,2,0,1,2 로 중복되지 않게 하려면 어떤 옵션이 필요한가요?

<details>
<summary>정답 보기</summary>

1. `inner` 는 양쪽에 모두 `customerNumber` 가 있는 행만 남겨 데이터가 줄어들고, `left` 는 왼쪽(`orders`) 전부를 유지하므로 매칭 안 되는 고객 정보는 `NaN` 으로 채워진 채 행 수가 보존됩니다.
2. `how="right"`(customers를 오른쪽에 둔 경우) 또는 `how="outer"`. 주문 없는 고객은 주문 관련 컬럼이 `NaN` 으로 채워집니다.
3. `pd.concat([q1, q2], ignore_index=True)` — `ignore_index=True` 가 인덱스를 0부터 새로 매깁니다.
</details>

---

## 헷갈리기 쉬운 포인트

- **concat vs merge**: concat은 "그냥 쌓기", merge는 "공통 키로 매칭해 잇기".
- **inner vs left**: inner는 교집합(행 줄어듦), left는 왼쪽 보존(안전). 행 수가 줄면 의도치 않은 inner를 의심.
- **merge vs join**: merge는 컬럼 키 중심, join은 인덱스 키 중심.
- **merge vs merge_asof**: 전자는 "정확히 같은 키", 후자는 "가장 가까운 키"(시계열에 필수, 사전 정렬 필요).
- **axis=0 vs axis=1 (concat)**: 0은 세로로 쌓기, 1은 가로로 붙이기(인덱스 기준).

## 연결되는 개념

- 이전 실습: [Python으로 푸는 선형대수 기초 문제](05-linear-algebra-practice-python.md) — NumPy로 벡터·행렬 계산 검증
- 큰 그림: [선형대수학, 머신러닝의 언어](01-linear-algebra-for-ml.md) — 병합으로 정돈한 데이터가 결국 벡터·행렬이 됨
- 더 찾아볼 키워드: `SQL JOIN`, `groupby`, `pivot_table`, `melt`, `데이터 정규화`, `결측치 처리`

## 셀프 체크

- [ ] concat의 `axis=0`/`axis=1` 차이를 안다.
- [ ] inner/left/right/outer 네 조인의 결과 차이를 설명할 수 있다.
- [ ] 컬럼명이 다를 때 `left_on`/`right_on` 을 쓸 수 있다.
- [ ] 세 개 이상 표를 체인 merge로 연결할 수 있다.
- [ ] `merge_asof` 가 시계열 근사 매칭임을 안다(정렬 필요).
- [ ] 조인 후 `NaN` 이 생기는 이유를 안다.

**복습 질문 및 답변**

- (기본) `concat` 과 `merge` 의 가장 큰 차이는?
  <details><summary>답</summary>concat은 공통 키 없이 단순히 이어붙이고, merge는 공통 키를 기준으로 매칭해 합칩니다.</details>
- (이해확인) `inner` 조인 후 데이터 행 수가 줄었습니다. 왜일까요?
  <details><summary>답</summary>inner는 양쪽에 모두 키가 있는 행만 남기기 때문입니다. 한쪽에만 있는 행은 버려집니다.</details>
- (응용) 각 주문에 대해 "그 주문 직전의 가장 최근 결제"를 붙이려면 어떤 함수를 어떤 옵션으로 써야 하나요?
  <details><summary>답</summary>`pd.merge_asof(..., direction="backward", by="customerNumber")`. 양쪽을 날짜 기준으로 정렬해 둬야 하고, `by` 로 같은 고객 안에서만 매칭합니다.</details>

## 한 줄 정리

> 분석의 시작은 흩어진 표를 올바르게 합치는 것이며, concat(쌓기)·merge(키 조인)·join(인덱스)·merge_asof(시계열)·combine_first/compare(보완·비교)를 상황에 맞게 골라 쓰는 것이 핵심이다.
