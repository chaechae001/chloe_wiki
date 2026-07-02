# 단계별 정규화 — 1NF·2NF·3NF 실습

> 앞 글에서 본 세 가지 이상 현상을, 이번엔 직접 없애봅니다. 공유 킥보드 테이블 하나를 단계별로 쪼개면서 "왜 이렇게 나누는지"를 코드 실행 결과로 확인합니다.

`정규화` `1NF` `2NF` `3NF` `원자값` `부분함수종속` `이행함수종속` `복합키` `외래키` `JOIN`

## 핵심요약

- 정규화는 한 번에 끝나지 않고 **1NF → 2NF → 3NF** 순서로 단계를 밟는다.
- **1NF**: 한 칸에 값 하나만(원자값). 여러 값이 들어 있으면 행을 나눈다.
- **2NF**: 복합키의 일부만으로 결정되는 속성(부분 함수 종속)을 떼어낸다.
- **3NF**: 키가 아닌 속성이 또 다른 속성을 결정하는 관계(이행 함수 종속)를 떼어낸다.
- 테이블을 나눠도 **JOIN으로 원본과 동일한 정보를 다시 합쳐 볼 수 있다.**
- 단계마다 "기본키가 무엇인가"를 먼저 정하면 다음에 뗄 속성이 보인다.

## 개념별 정리

### 1차 정규화 (1NF) — 값을 원자값으로

**1. 정의**
테이블의 모든 컬럼이 **더 이상 나눌 수 없는 하나의 값(원자값)** 만 갖도록 만드는 단계입니다.

**2. 왜 필요한가?**
한 칸에 값이 여러 개면 검색·집계·수정이 모두 꼬입니다. "이 칸에서 두 번째 시간만 바꿔줘" 같은 작업이 불가능하기 때문입니다.

**3. 예시**
`rental_time`에 두 개의 시간이 쉼표로 붙어 있던 것을 행으로 분리합니다.

```python
# 쉼표로 합쳐진 rental_time을 분리해 행을 나눈다
cursor = conn.execute("SELECT * FROM customer_unnormalized")
for row in cursor.fetchall():
    cid, name, times, loc, brand, year, ppm, bp, comp = row
    for t in [t.strip() for t in times.split(",")]:
        conn.execute("INSERT INTO customer_1nf VALUES (?,?,?,?,?,?,?,?,?)",
                     (cid, name, t, loc, brand, year, ppm, bp, comp))
```

```text
After (1NF 적용):
  customer_id customer_name          rental_time
0       kmax6           김민준  2020-08-20 13:01:02
1       kmax6           김민준  2020-09-01 20:39:52
2  freeman123           박서준  2021-01-05 09:15:00
3     flykite           이서연  2020-11-11 22:01:30
```

`kmax6`의 두 대여가 별도 행으로 나뉘었습니다. 이제 모든 칸이 값 하나씩만 갖습니다. 이 단계의 기본키는 `(고객 ID, 대여 시간)` **복합키**가 됩니다. 한 고객이 여러 번 빌릴 수 있으니 고객 ID 하나만으로는 행을 구분할 수 없기 때문입니다.

**4. 헷갈리기 쉬운 점**
1NF를 했더니 오히려 `김민준`이 두 번 반복되어 "중복이 늘어난 것 아닌가?" 싶습니다. 맞습니다. 1NF는 원자값만 보장할 뿐, 중복으로 인한 갱신·삭제 이상은 아직 남아 있습니다. 그래서 2NF가 필요합니다.

**5. 한 줄 정리**
1NF는 "한 칸에 하나의 값"을 보장하는 단계입니다.

> 비유: 한 칸에 전화번호 두 개를 적어두면 정렬도 검색도 안 되니, 한 줄에 하나씩 다시 적는 것.

### 2차 정규화 (2NF) — 부분 함수 종속 제거

**1. 정의**
1NF를 만족하면서, **복합키의 일부만으로 결정되는 속성(부분 함수 종속)을 없애** 모든 비(非)키 속성이 키 전체에 완전 종속되도록 만드는 단계입니다.

**2. 왜 필요한가?**
`customer_name`은 `customer_id` 하나만 알면 정해지는데, 키는 `(customer_id, rental_time)`입니다. 즉 키의 일부에만 매달려 있어 대여할 때마다 이름이 중복 저장됩니다. 이걸 떼어내면 갱신·삭제 이상이 줄어듭니다.

**3. 예시**
고객 정보를 별도 테이블로 분리합니다.

```python
conn.executescript("""
CREATE TABLE customer_2nf (
    customer_id   TEXT PRIMARY KEY,
    customer_name TEXT NOT NULL
);
CREATE TABLE rental_2nf (
    customer_id TEXT, rental_time TEXT, rental_location TEXT,
    brand TEXT, model_year INTEGER,
    price_per_min INTEGER, basic_price INTEGER, company TEXT,
    PRIMARY KEY (customer_id, rental_time),
    FOREIGN KEY (customer_id) REFERENCES customer_2nf(customer_id)
);
""")
conn.execute("INSERT INTO customer_2nf SELECT DISTINCT customer_id, customer_name FROM customer_1nf")
```

```text
customer 테이블 (분리됨)
  customer_id customer_name
0       kmax6           김민준
1  freeman123           박서준
2     flykite           이서연
```

이제 고객 이름은 `customer_2nf`에 **한 번만** 저장됩니다. 대여 테이블은 `customer_id`를 **외래키**로 참조합니다. 덕분에 대여 이력이 없어도 고객만 먼저 등록할 수 있어 삽입 이상도 해소됩니다.

**4. 헷갈리기 쉬운 점**
2NF는 **기본키가 복합키일 때만 의미가 있습니다.** 키가 단일 컬럼이면 부분 종속이 생길 수 없으므로 1NF면 자동으로 2NF입니다.

**5. 한 줄 정리**
2NF는 "키의 일부에만 매달린 속성"을 떼어 별도 테이블로 보내는 단계입니다.

> 비유: 고객 명부와 대여 장부를 한 권에 섞어 쓰다가, 명부와 장부를 따로 분리하는 것.

### 3차 정규화 (3NF) — 이행 함수 종속 제거

**1. 정의**
2NF를 만족하면서, **키가 아닌 속성이 또 다른 속성을 결정하는 관계(이행 함수 종속)** 를 없애는 단계입니다.

**2. 왜 필요한가?**
`rental_2nf`에서 `brand`(키 아님)가 `company`, `price_per_min`을 결정합니다(`대여기록 → brand → company`). 이러면 `boardkick`의 요금이 바뀔 때 그 브랜드를 빌린 모든 행을 고쳐야 합니다. 브랜드 정보를 한곳에 모으면 한 번만 고치면 됩니다.

**3. 예시**
브랜드(킥보드) 정보를 별도 테이블로 분리합니다.

```python
conn.executescript("""
CREATE TABLE kickboard_3nf (
    brand TEXT PRIMARY KEY, model_year INTEGER,
    price_per_min INTEGER, basic_price INTEGER, company TEXT
);
CREATE TABLE rental_3nf (
    customer_id TEXT, rental_time TEXT, rental_location TEXT, brand TEXT,
    PRIMARY KEY (customer_id, rental_time),
    FOREIGN KEY (customer_id) REFERENCES customer_2nf(customer_id),
    FOREIGN KEY (brand)       REFERENCES kickboard_3nf(brand)
);
""")
```

```text
kickboard 테이블 (분리됨)
       brand  model_year  price_per_min  basic_price     company
0  boardkick        2015            100         1000       rideon
1     willgo        2018            110          950  everythere
```

이제 브랜드별 요금·회사명은 `kickboard_3nf`에 딱 한 줄씩만 존재합니다. 대여 테이블에는 `brand`만 외래키로 남깁니다.

**4. 헷갈리기 쉬운 점**
3NF의 대상은 "비키 → 비키" 종속입니다. 키가 직접 결정하는 속성(`키 → 속성`)은 정상이며 떼어내지 않습니다. 떼는 것은 어디까지나 **중간 다리 역할을 하는 비키 속성**입니다.

**5. 한 줄 정리**
3NF는 "키가 아닌데 다른 걸 결정하는 속성"을 떼어 마스터 테이블로 보내는 단계입니다.

> 비유: 장부마다 브랜드 요금을 매번 적지 말고, 브랜드 요금표를 따로 만들어 참조만 하는 것.

## 코드로 보기 — 나눈 테이블을 JOIN으로 다시 합치기

```python
df = pd.read_sql_query("""
    SELECT c.customer_id, c.customer_name, r.rental_time, r.rental_location,
           r.brand, k.model_year, k.price_per_min, k.basic_price, k.company
    FROM rental_3nf r
    JOIN customer_2nf  c ON r.customer_id = c.customer_id
    JOIN kickboard_3nf k ON r.brand       = k.brand
    ORDER BY r.rental_time
""", conn)
print(df)
```

```text
  customer_id customer_name          rental_time rental_location      brand  model_year  price_per_min  basic_price     company
0       kmax6           김민준  2020-08-20 13:01:02     서울시 강남구 역삼동  boardkick        2015            100         1000       rideon
1       kmax6           김민준  2020-09-01 20:39:52     서울시 강남구 역삼동  boardkick        2015            100         1000       rideon
2     flykite           이서연  2020-11-11 22:01:30     서울시 동작구 대방동     willgo        2018            110          950  everythere
3  freeman123           박서준  2021-01-05 09:15:00     서울시 관악구 신림동  boardkick        2015            100         1000       rideon
```

**코드목적**
세 개로 나눈 테이블(`customer`, `rental`, `kickboard`)을 다시 연결해, 정규화 전 원본과 똑같은 정보를 볼 수 있는지 검증합니다.

**해석**
나눠도 잃어버린 정보가 없습니다. 평소에는 분리된 채로 안전하게 저장하고, 필요할 때만 JOIN으로 합쳐서 보면 됩니다. 정규화는 "정보를 버리는 것"이 아니라 "안전하게 쪼개 두는 것"임을 보여줍니다.

**실무 연결**
실제 서비스의 화면(예: 대여 내역 페이지)은 대부분 이렇게 여러 테이블을 JOIN해 만든 결과입니다. 데이터는 분리해 저장하고, 보여줄 때 조립하는 패턴이 기본입니다.

## 직접 해보기

1. 1NF 테이블의 기본키가 왜 `(customer_id, rental_time)` 복합키여야 하는지 한 문장으로 설명해보세요.
2. 2NF에서 분리된 `customer` 테이블의 기본키와, `rental` 테이블이 그걸 참조하는 외래키를 각각 적어보세요.
3. 3NF의 `kickboard` 테이블에서 `brand`가 결정하는 속성을 모두 나열해보세요.

<details>
<summary>답안 보기</summary>

1. 한 고객이 여러 번 빌릴 수 있어 `customer_id`만으로는 행을 구분할 수 없고, 대여 시간까지 합쳐야 유일하게 식별되기 때문.
2. `customer` 기본키 = `customer_id`, `rental`의 외래키 = `customer_id`(→ customer 참조).
3. `model_year`, `price_per_min`, `basic_price`, `company`.
</details>

## 헷갈리기 쉬운 포인트

- **1NF vs 2NF**: 1NF는 *값의 모양*(원자값) 문제, 2NF는 *키와의 관계*(부분 종속) 문제.
- **2NF vs 3NF**: 2NF는 *키의 일부*에 매달린 속성 제거, 3NF는 *키가 아닌 속성*이 결정하는 속성 제거.
- **PRIMARY KEY vs FOREIGN KEY**: PK는 "이 테이블에서 행을 구분하는 열쇠", FK는 "다른 테이블의 PK를 가리키는 연결 고리".

## 연결되는 개념

- 이전 글: [이상 현상과 함수 종속](01-anomalies-and-functional-dependency.md)
- 다음 글: [정규화 한눈에 정리 — BCNF·4NF·5NF와 역정규화](03-normalization-summary-denormalization.md)
- 더 찾아볼 키워드: `정규형`, `참조 무결성`, `서로게이트 키(surrogate key)`

## 셀프 체크

- [ ] 1NF·2NF·3NF가 각각 무엇을 제거하는지 말할 수 있다.
- [ ] 복합키에서 부분 함수 종속을 찾을 수 있다.
- [ ] 이행 함수 종속의 예를 들 수 있다.
- [ ] 나눈 테이블을 JOIN으로 합칠 수 있음을 이해한다.

**복습 질문 및 답변**

- (기본) 1NF의 조건은? → 모든 컬럼이 원자값(나눌 수 없는 하나의 값)을 가질 것.
- (이해확인) 2NF에서 `customer_name`을 분리한 이유는? → `customer_id`만으로 결정되는 부분 함수 종속이라 대여마다 중복 저장되기 때문.
- (응용) `학번 → 학과 → 학과사무실` 관계가 있는 테이블은 몇 차 정규화가 필요할까? → 학과가 비키 속성으로 학과사무실을 결정하는 이행 종속이므로 3NF가 필요.

## 한 줄 정리

> 정규화는 기본키를 정하고, 키와의 종속 관계를 보며 테이블을 1NF→2NF→3NF로 쪼개는 과정이며, 나눈 뒤에도 JOIN으로 원본을 그대로 되살릴 수 있습니다.
