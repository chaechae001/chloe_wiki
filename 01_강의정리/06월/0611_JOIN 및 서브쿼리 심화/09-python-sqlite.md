# Python + SQLite 연동과 시각화 — 추출 → 처리 → 그림으로

> SQL로 데이터를 잘 뽑았다면, 그 다음은? DB는 '저장소'일 뿐, 실제 분석과 서비스는 프로그래밍 언어에서 합니다. SQL 결과를 Python으로 가져와 표·그래프로 바꾸는 한 사이클을 따라가 봅니다.

`Python` `sqlite3` `matplotlib` `CRUD` `데이터연동` `시각화`

## 핵심요약

- DB는 저장소 역할이 중심이고, 실제 분석·시각화·서비스는 **프로그래밍 언어에서 DB에 접속 → 쿼리 실행 → 결과 처리**로 이뤄진다.
- Python 표준 라이브러리 **`sqlite3`**로 별도 설치 없이 SQLite DB를 다룰 수 있다.
- 기본 흐름은 **`connect()` → `cursor.execute(sql)` → `fetchall()`** 세 단계의 반복이다.
- 데이터를 바꾸는 작업(INSERT·UPDATE·DELETE)은 끝에 **`commit()`**으로 확정해야 한다.
- SQL로 뽑은 결과를 **`matplotlib`**의 `fig, ax = plt.subplots()` 패턴으로 그래프화한다.
- 같은 JOIN·서브쿼리 SQL을 그대로 Python 안에서 실행할 수 있다 — SQL 지식이 곧바로 재사용된다.

## 개념별 정리

### 왜 Python으로 연결하는가

**1. 정의**
DB에 저장된 데이터를 프로그래밍 언어에서 불러와 가공·분석·시각화하는 작업을 "DB 연동"이라 한다.

**2. 왜 필요한가?**
DB만으로는 표 형태 조회까지가 한계다. 그래프 그리기, 머신러닝, 웹 서비스 화면 구성 등은 모두 언어 쪽에서 한다. 그래서 "DB에서 필요한 만큼 SQL로 뽑고 → 언어로 받아 처리"하는 흐름이 표준이다.

**3. 실습 흐름(반복 패턴)**
1. DB 연결
2. 쿼리 작성·실행
3. 결과 처리(표·차트 등)

이 세 단계가 데이터 작업 내내 반복된다.

**4. 헷갈리기 쉬운 점**
"SQL을 잘하면 Python은 따로 또 배워야 하나?"라고 생각하기 쉽지만, 연동의 핵심은 **SQL 문자열을 그대로 넘기는 것**이다. 앞서 배운 JOIN·서브쿼리 SQL이 그대로 쓰인다.

**5. 한 줄 정리**
DB는 창고, 언어는 작업장 — 창고에서 꺼내(SQL) 작업장에서 가공(Python)한다.

> 비유: 도서관(DB)에서 필요한 책만 대출증(쿼리)으로 빌려 와, 집(Python)에서 읽고 정리하는 것.

### 기본 연결 패턴: connect → execute → fetchall

**1. 정의**
`sqlite3.connect()`로 접속하고, 커서로 `execute()`해 쿼리를 보내고, `fetchall()`로 결과를 받는 3단계 패턴이다.

**2. 왜 필요한가?**
모든 조회의 뼈대가 이 셋이다. 한 번 함수로 감싸 두면 어떤 SQL이든 재사용할 수 있다.

**3. 예시 (재사용 함수)**
```python
import sqlite3

def run_query(db_path, sql, params=()):
    """SQLite 쿼리 실행 후 (컬럼명 리스트, 행 리스트) 반환"""
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row      # dict처럼 컬럼명으로 접근 가능
    cur = conn.cursor()
    cur.execute(sql, params)
    rows = cur.fetchall()
    columns = [desc[0] for desc in cur.description] if cur.description else []
    conn.close()
    return columns, rows
```

**4. 헷갈리기 쉬운 점**
`row_factory = sqlite3.Row`를 설정하면 결과를 `row['컬럼명']`처럼 이름으로 꺼낼 수 있어 훨씬 읽기 쉽다. 또 작업이 끝나면 `conn.close()`로 연결을 닫아야 한다. ("closed database" 오류가 나면 보통 연결이 닫힌 뒤 다시 쓰려 한 경우 — 연결 코드를 다시 실행하면 풀린다.)

**5. 한 줄 정리**
connect → execute → fetchall, 이 세 줄이 모든 조회의 기본 골격이다.

### 데이터를 바꾸면 commit — CRUD

**1. 정의**
Create(생성)·Read(조회)·Update(수정)·Delete(삭제)를 묶어 CRUD라 한다. SQL의 INSERT·SELECT·UPDATE·DELETE에 대응한다.

**2. 왜 필요한가?**
조회만이 아니라 데이터를 넣고 고치고 지우는 일까지 해야 실제 애플리케이션이 된다.

**3. 예시 (수정 후 확정)**
```python
# UPDATE: 음료(category_id=1) 가격 10% 인상
cur.execute("UPDATE products SET price = price * 1.1 WHERE category_id = 1")
conn.commit()        # ★ 변경은 commit으로 확정

cur.execute("SELECT product_name, price FROM products WHERE category_id = 1")
for row in cur.fetchall():
    print(row)
# ('아메리카노', 4950.0)
# ('녹차', 4400.0)
```

**4. 헷갈리기 쉬운 점**
SELECT는 `commit()`이 필요 없지만, **데이터를 바꾸는 INSERT·UPDATE·DELETE는 반드시 `commit()`**해야 실제로 반영된다. commit을 빼먹으면 변경이 사라진다(롤백).

**5. 한 줄 정리**
조회는 그냥, 변경은 `commit()` — 바꿨으면 확정 도장을 찍는다.

## 코드로 보기 — SQL 결과를 그래프로

아래는 `classicmodels` 샘플 DB에서 국가별 고객 수를 뽑아 막대 그래프로 그리는 흐름이다(노트북 실행 결과 기준으로 복원한 쿼리).

```python
# 1) 국가별 고객 수 조회 (GROUP BY)
sql_customers = """
SELECT country, COUNT(*) AS customer_count
FROM customers
GROUP BY country
ORDER BY customer_count DESC
LIMIT 8
"""
cols, rows = run_query(CLASSIC_DB, sql_customers)
# 결과 예: USA 36, Germany 13, France 12, Spain 7, UK 5 ...

# 2) matplotlib로 막대그래프
import matplotlib.pyplot as plt

countries = [row['country'] for row in rows]
counts    = [row['customer_count'] for row in rows]

fig, ax = plt.subplots(figsize=(9, 4))
ax.bar(countries, counts, color='steelblue', edgecolor='white')
ax.set_title('국가별 고객 수 (상위 8개국)')
ax.set_xlabel('국가')
ax.set_ylabel('고객 수')
ax.tick_params(axis='x', rotation=35)
fig.tight_layout()
plt.show()
```

**코드목적**
SQL로 집계한 "국가별 고객 수"를 Python으로 받아, 한눈에 비교되는 막대그래프로 시각화한다.

**해석**
1단계에서 `GROUP BY`로 국가별 인원을 세고 상위 8개국만 가져온다. 2단계에서 결과의 국가 이름과 수치를 각각 리스트로 분리한 뒤, `fig, ax = plt.subplots()`로 도화지를 펴고 `ax.bar(...)`로 막대를 그린다. 표(숫자)였던 결과가 그림(길이)으로 바뀌어 "USA가 압도적으로 많다"가 즉시 보인다.

**실무 연결**
"SQL로 집계 → 리스트로 분리 → `plt.subplots()`로 시각화"는 데이터 분석 보고서의 가장 흔한 한 사이클이다. 제품 라인별 매출(JOIN + GROUP BY)을 가로 막대로 그리는 등, 쿼리만 바꾸면 같은 틀로 무한히 응용된다.

## 직접 해보기

> 아래는 미니 쇼핑몰 연습 DB(`day04_shop.db`: categories·products·customers·orders·order_items)를 만들고 다루는 연습입니다. 노트북 실행 결과로 정답 쿼리를 복원했습니다.

1. **JOIN**: 고객·주문·주문상세·상품을 이어 "누가 무엇을 몇 개 샀고 합계가 얼마인지"를 출력해 보세요.
   ```sql
   SELECT c.customer_name, o.order_id, p.product_name, oi.quantity,
          p.price * oi.quantity AS line_total
   FROM orders o
   JOIN customers   c  ON o.customer_id = c.customer_id
   JOIN order_items oi ON o.order_id    = oi.order_id
   JOIN products    p  ON oi.product_id = p.product_id
   ORDER BY o.order_id;
   ```

2. **JOIN + GROUP BY**: 카테고리별 총 판매 수량을 많은 순으로 집계해 보세요.
   ```sql
   SELECT cat.category_name, SUM(oi.quantity) AS total_qty
   FROM order_items oi
   JOIN products   p   ON oi.product_id = p.product_id
   JOIN categories cat ON p.category_id = cat.category_id
   GROUP BY cat.category_name
   ORDER BY total_qty DESC;
   ```

3. **서브쿼리(IN)**: '음료'(category_id=1) 상품이 포함된 주문의 번호·고객명·날짜를 조회해 보세요.
   ```sql
   SELECT DISTINCT o.order_id, c.customer_name, o.order_date
   FROM orders o
   JOIN customers c ON o.customer_id = c.customer_id
   WHERE o.order_id IN (
       SELECT oi.order_id
       FROM order_items oi
       WHERE oi.product_id IN (
           SELECT product_id FROM products WHERE category_id = 1
       )
   )
   ORDER BY o.order_id;
   ```

## 헷갈리기 쉬운 포인트

- **SELECT vs 변경 작업**: 조회는 `commit()` 불필요, INSERT·UPDATE·DELETE는 `commit()` 필수.
- **`fetchall()` vs `fetchone()`**: 전체를 한 번에 받으면 `fetchall()`, 한 행씩이면 `fetchone()`.
- **`row[0]` vs `row['컬럼명']`**: `row_factory = sqlite3.Row` 설정 시 컬럼명 접근이 가능해 가독성이 좋다.
- **연결 닫힘 오류("closed database")**: 연결을 닫은 뒤 다시 쓰면 발생 — 연결 코드를 다시 실행하면 된다.

## 연결되는 개념

- 이전 글들: [① JOIN 기초](01-join-basics-inner.md) ~ [⑧ 뷰(VIEW)](08-view.md) — 여기서 배운 SQL을 그대로 Python에서 실행
- 함께 보면 좋은: [⑥ 서브쿼리 반환 형태](06-subquery-return-shape.md) — 연습 3번의 IN 서브쿼리
- 더 찾아볼 키워드: `pandas.read_sql`, `matplotlib`, `with 문(컨텍스트 매니저)`, `파라미터 바인딩(?)`

## 셀프 체크

- [ ] "DB에서 뽑아 언어로 가공"하는 흐름을 설명할 수 있다.
- [ ] `connect → execute → fetchall` 3단계를 안다.
- [ ] 변경 작업에 `commit()`이 필요한 이유를 안다.
- [ ] SQL 결과를 리스트로 분리해 `plt.subplots()`로 그릴 수 있다.
- [ ] 앞서 배운 JOIN/서브쿼리 SQL을 Python에서 그대로 쓸 수 있음을 안다.

**복습 질문 및 답변**

- (기본) Python에서 SQLite를 쓰려면 별도 설치가 필요한가요?
  → 아닙니다. `sqlite3`는 Python 표준 라이브러리라 바로 `import`해 쓸 수 있습니다.
- (이해 확인) `UPDATE` 후 `commit()`을 빼먹으면 어떻게 되나요?
  → 변경이 확정되지 않아 사실상 적용되지 않습니다(연결을 닫으면 사라집니다).
- (응용) SQL 집계 결과를 막대그래프로 그리는 일반적인 순서는?
  → 쿼리 실행 → 결과를 라벨·값 리스트로 분리 → `fig, ax = plt.subplots()` → `ax.bar(...)` → `plt.show()`.

## 한 줄 정리

> DB에서 SQL로 뽑아 Python으로 받아 처리·시각화하는 한 사이클(connect → execute → fetchall, 변경 시 commit)이 데이터 작업의 기본 리듬이다.
