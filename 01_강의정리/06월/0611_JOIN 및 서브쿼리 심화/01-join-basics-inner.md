# JOIN 기초 — 두 표를 잇는 원리

> "고객 표에는 이름만 있고, 주문 표에는 번호만 있어요. 둘을 어떻게 한 화면에서 같이 보죠?" — 이 질문의 답이 JOIN입니다.

`JOIN` `EQUI JOIN` `Non-EQUI JOIN` `INNER JOIN` `ON절` `기본키-외래키`

## 핵심요약

- 데이터는 보통 여러 표에 흩어져 저장되고, **JOIN**은 그 표들을 이어 붙여 한 결과로 만든다.
- 연결 방식을 연산자로 나누면 **EQUI JOIN(= 사용)**과 **Non‑EQUI JOIN(>, <, BETWEEN 등 사용)**으로 갈린다.
- 가장 많이 쓰는 건 **INNER JOIN**으로, 양쪽 표에서 **조건이 맞는 행만** 남긴다(교집합).
- `INNER` 키워드는 생략 가능 — 그냥 `JOIN`이라고만 써도 INNER JOIN이다.
- **ON절**로 조인 조건을 직접 지정하면, 두 표의 컬럼 이름이 달라도 연결할 수 있다.
- 대부분의 EQUI JOIN은 **기본키–외래키 관계**를 따라가지만, 반드시 그런 건 아니다.

## 개념별 정리

### JOIN이란 무엇인가

**1. 정의**
두 개 이상의 테이블을 특정 조건으로 연결·결합해 하나의 결과로 출력하는 것이다.

**2. 왜 필요한가?**
현실 데이터는 중복을 줄이려고 표를 나눠 저장한다. 고객 정보를 주문마다 반복 저장하지 않고, 주문 표에는 고객 번호만 적어 둔다. 그래서 "이 주문을 한 고객의 이름과 이메일"을 보려면 두 표를 이어야 한다. 데이터 분석, 리포트, 서비스 화면 구성 거의 전부가 JOIN 위에서 돌아간다.

**3. 예시**
앨범 표와 아티스트 표를 연결해 "앨범 제목 + 아티스트 이름"을 뽑는다. 아래는 가장 원초적인 형태인 WHERE절 등가 조인이다.

```sql
SELECT al.Title AS AlbumTitle,
       ar.Name  AS ArtistName
FROM albums al, artists ar
WHERE al.ArtistId = ar.ArtistId
ORDER BY al.Title;
```

**4. 헷갈리기 쉬운 점**
JOIN은 표를 "위아래로 쌓는" 게 아니라 "좌우로 붙이는" 작업이다. 행을 이어 붙여 세로로 길게 만드는 건 `UNION`이고, JOIN은 한 행 옆에 다른 표의 컬럼을 붙여 가로로 넓게 만든다.

**5. 한 줄 정리**
JOIN은 흩어진 표를 조건으로 이어 붙여 하나의 넓은 표로 만드는 일이다.

> 비유: 출석부(이름·학번)와 성적표(학번·점수)를 학번 기준으로 나란히 놓고 한 줄씩 맞춰 보는 것.

### EQUI JOIN vs Non-EQUI JOIN

**1. 정의**
연결 조건에 어떤 연산자를 쓰느냐로 나눈 분류다. **EQUI JOIN**은 등호(`=`)로 "정확히 같은 값"을 잇고, **Non‑EQUI JOIN**은 `>`, `>=`, `<`, `<=`, `BETWEEN` 같은 비교 연산자로 "범위/대소" 관계를 잇는다.

**2. 왜 필요한가?**
대부분의 연결은 "고객번호가 같은 행끼리"처럼 등가 조건이라 EQUI JOIN이다. 하지만 "점수가 어느 등급 구간에 드는가"처럼 정확히 일치하지 않고 범위로 매칭해야 할 때 Non‑EQUI JOIN이 쓰인다.

**3. 예시 (Non-EQUI: 범위로 등급 매기기)**
트랙 단가를 구간별 등급으로 분류한다. 등급 기준표와 범위(`<=`)로 연결하는 발상이며, 실무에선 아래처럼 CASE로도 표현한다.

```sql
SELECT Name,
       UnitPrice,
       CASE
           WHEN UnitPrice <= 0.99 THEN 'Standard'
           WHEN UnitPrice <= 1.29 THEN 'Premium'
           ELSE 'High'
       END AS PriceGrade
FROM tracks
ORDER BY UnitPrice DESC;
```

**4. 헷갈리기 쉬운 점**
"EQUI = 좋은 조인, Non‑EQUI = 나쁜 조인"이 아니다. 둘은 목적이 다를 뿐이다. 다만 Non‑EQUI JOIN은 매칭되는 행이 많아 결과가 커지기 쉬우니 범위 조건을 신중히 잡아야 한다.

**5. 한 줄 정리**
정확히 같은 값으로 이으면 EQUI, 크고 작음·범위로 이으면 Non‑EQUI다.

### INNER JOIN (ON절)

**1. 정의**
조인 조건을 **만족하는 행만** 양쪽에서 골라 결합하는 조인이다. 양쪽 모두에 짝이 있어야 결과에 남으므로 흔히 "교집합"으로 설명한다.

**2. 왜 필요한가?**
"주문이 실제로 있는 고객만", "장르가 지정된 트랙만"처럼 **양쪽에 데이터가 다 있는 경우**만 보고 싶을 때 쓴다. 가장 기본이자 가장 자주 쓰는 조인이다.

**3. 예시 (트랙 ↔ 장르)**
`ON`으로 조인 조건을 직접 지정하면, 두 표의 연결 컬럼 이름이 달라도 연결할 수 있다.

```sql
SELECT t.Name   AS TrackName,
       g.Name   AS GenreName
FROM tracks t
INNER JOIN genres g ON t.GenreId = g.GenreId
ORDER BY g.Name;
```

**4. 헷갈리기 쉬운 점**
`JOIN`만 쓰면 자동으로 INNER JOIN이다. `INNER`는 생략 가능한 장식이라고 봐도 된다. 또 옛 문법인 `FROM a, b WHERE a.x = b.y`도 결과적으로 INNER JOIN이지만, 조인 조건과 일반 필터가 WHERE에 뒤섞여 읽기 어렵다. 가독성과 실수 방지를 위해 **`JOIN ... ON ...` 형태를 권장**한다.

**5. 한 줄 정리**
INNER JOIN은 양쪽에 짝이 있는 행만 남기는, 가장 기본적인 표 결합이다.

> 비유: 짝짓기 게임에서 파트너가 양쪽 다 있는 사람만 무대에 남기는 것.

## 코드로 보기 — 세 표를 한 번에 잇기

```sql
SELECT i.InvoiceId,
       t.Name   AS TrackName,
       al.Title AS AlbumTitle,
       (ii.UnitPrice * ii.Quantity) AS SubTotal
FROM invoices i
INNER JOIN invoice_items ii ON i.InvoiceId = ii.InvoiceId
INNER JOIN tracks         t  ON ii.TrackId  = t.TrackId
INNER JOIN albums         al ON t.AlbumId   = al.AlbumId
ORDER BY SubTotal DESC
LIMIT 10;
```

**코드목적**
주문(invoices) → 주문 항목(invoice_items) → 트랙(tracks) → 앨범(albums)을 차례로 이어, "어떤 주문에서 어떤 트랙이 얼마치 팔렸는가"를 한 줄로 본다.

**해석**
JOIN은 두 표씩 차례로 붙여 나간다. 1단계로 주문과 주문항목을 잇고, 2단계로 트랙을, 3단계로 앨범을 붙인다. 마지막에 `단가 × 수량`을 계산한 `SubTotal`로 정렬해 매출이 큰 항목 10개를 본다. 표가 늘어도 "키가 맞는 컬럼끼리 ON으로 잇는다"는 원리는 동일하다.

**실무 연결**
주문·상품·고객 표를 잇는 이런 다중 JOIN은 매출 리포트, 대시보드, 추천 데이터 준비 등 거의 모든 분석의 1단계다. 한 번에 다 외우기보다 "표 두 개씩 이어 붙인다"는 감각을 먼저 잡는 게 중요하다.

## 직접 해보기

1. 앨범(albums)과 아티스트(artists)를 INNER JOIN(ON절)으로 연결해 앨범 제목과 아티스트 이름을 조회하고, 앨범 제목 오름차순으로 정렬해 보세요.
2. 트랙(tracks)·앨범(albums)·아티스트(artists) 세 표를 이어 트랙명·앨범 제목·아티스트 이름을 상위 10행만 조회해 보세요.
3. 고객(customers)과 주문(invoices)을 INNER JOIN하고, 국가가 `'Brazil'`인 고객의 이름·이메일·주문 날짜·총액을 조회해 보세요. (조인 후 `WHERE`로 국가를 거는 연습)

## 헷갈리기 쉬운 포인트

- **`JOIN` vs `INNER JOIN`**: 완전히 같다. `INNER`는 생략 가능.
- **`ON` vs `WHERE`**: `ON`은 "어떻게 이을지(조인 조건)", `WHERE`는 "이은 다음 무엇을 남길지(필터)". INNER JOIN에선 결과가 비슷해 보여도, OUTER JOIN에서는 이 차이가 결정적이다([③ OUTER JOIN](03-outer-join.md) 참고).
- **EQUI vs Non‑EQUI**: 등호로 이으면 EQUI, 부등호·BETWEEN으로 이으면 Non‑EQUI.

## 연결되는 개념

- 다음 글: [② USING · NATURAL · CROSS JOIN](02-using-natural-cross-join.md) — 같은 컬럼으로 잇는 간편 문법과 모든 조합 만들기
- 이어지는 글: [③ OUTER JOIN](03-outer-join.md) — 짝 없는 행도 남기는 방법
- 더 찾아볼 키워드: `기본키(PK)`, `외래키(FK)`, `정규화`, `카티션 곱`

## 셀프 체크

- [ ] JOIN이 "표를 가로로 잇는" 작업임을 설명할 수 있다.
- [ ] EQUI JOIN과 Non‑EQUI JOIN을 연산자 기준으로 구분할 수 있다.
- [ ] INNER JOIN이 "양쪽에 짝이 있는 행"만 남긴다는 것을 안다.
- [ ] `ON`으로 컬럼명이 달라도 조인할 수 있음을 안다.
- [ ] 세 개 이상의 표를 차례로 JOIN할 수 있다.

**복습 질문 및 답변**

- (기본) `INNER JOIN`에서 `INNER`를 생략하면 어떻게 되나요?
  → 그냥 `JOIN`이 되며, 동작은 동일한 INNER JOIN입니다.
- (이해 확인) 두 표의 연결 컬럼 이름이 서로 다를 때(`CLASS` vs `ID`)도 조인할 수 있나요?
  → 네. `ON 테이블1.컬럼 = 테이블2.컬럼`처럼 ON절에 직접 조건을 적으면 됩니다.
- (응용) "장르가 지정된 트랙만" 보고 싶다면 INNER JOIN이 적절한 이유는?
  → INNER JOIN은 양쪽에 짝이 있는 행만 남기므로, 장르가 없는(매칭 안 되는) 트랙은 자동으로 제외됩니다.

## 한 줄 정리

> JOIN은 흩어진 표를 조건으로 이어 붙이는 것이고, 그 기본은 양쪽에 짝이 있는 행만 남기는 INNER JOIN이다.
