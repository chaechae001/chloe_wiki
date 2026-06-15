# OUTER JOIN — 한쪽을 다 남기기, 그리고 "없는 데이터" 찾기

> "주문을 한 번도 안 한 고객은 누구지?" — INNER JOIN으로는 절대 못 찾습니다. 짝이 없는 행은 INNER JOIN이 버리니까요. 이럴 때가 OUTER JOIN의 무대입니다.

`OUTER JOIN` `LEFT JOIN` `RIGHT JOIN` `FULL OUTER JOIN` `IS NULL` `기준테이블`

## 핵심요약

- **OUTER JOIN**은 교집합에 더해, **한쪽 표에만 있는 행(짝 없는 행)까지** 결과에 포함한다.
- 짝이 없어 채울 값이 없는 칸은 **NULL**로 표시된다.
- **LEFT JOIN**은 왼쪽 표를 전부 남기고, **RIGHT JOIN**은 오른쪽 표를 전부 남긴다.
- **FULL OUTER JOIN**은 양쪽 표를 모두 남긴다.
- "주문 안 한 고객", "앨범 없는 아티스트"처럼 **없는 것 찾기**는 `LEFT JOIN + WHERE 오른쪽키 IS NULL` 패턴이 정석.
- OUTER JOIN에서는 **기준 테이블 선정**이 가장 중요하다 — "전체를 다 보고 싶은 표"를 기준(왼쪽)에 둔다.

## 개념별 정리

### OUTER JOIN과 NULL

**1. 정의**
두 표의 교집합(짝이 맞는 행)에 더해, **한쪽에만 존재하는 행도 포함**시켜 조회하는 조인이다. 짝이 없어 비는 컬럼은 NULL로 채워진다.

**2. 왜 필요한가?**
"전체 고객 목록을 보되, 주문이 있으면 같이 보고 없으면 빈칸으로 두고 싶다" 같은 요구가 많기 때문이다. INNER JOIN은 주문 없는 고객을 통째로 버리지만, OUTER JOIN은 남겨 둔다.

**3. 예시 (모든 아티스트 + 있으면 앨범)**
```sql
SELECT ar.Name   AS ArtistName,
       al.Title  AS AlbumTitle
FROM artists ar
LEFT JOIN albums al ON ar.ArtistId = al.ArtistId
ORDER BY ar.Name;
```
앨범이 없는 아티스트도 결과에 남고, 그 행의 `AlbumTitle`은 NULL이 된다.

**4. 헷갈리기 쉬운 점**
NULL은 "0"이나 "빈 문자열"이 아니라 "값이 없음" 그 자체다. 그래서 `= NULL`로는 비교가 안 되고 반드시 `IS NULL` / `IS NOT NULL`을 써야 한다.

**5. 한 줄 정리**
OUTER JOIN은 짝 없는 행까지 NULL로 채워 함께 보여 주는 조인이다.

### LEFT / RIGHT / FULL OUTER JOIN

**1. 정의**
어느 쪽 표를 "전부 남길지" 정하는 방향 선택이다. **LEFT**는 왼쪽(FROM에 먼저 쓴 표) 전부, **RIGHT**는 오른쪽 전부, **FULL**은 양쪽 전부를 남긴다.

**2. 왜 필요한가?**
"전체를 빠짐없이 보고 싶은 표"가 어느 쪽인지에 따라 방향을 고른다. 보통 사람들은 LEFT JOIN을 기준 테이블을 왼쪽에 두고 쓴다. RIGHT JOIN은 테이블 순서만 바꾸면 LEFT JOIN과 같아서, 실무에선 LEFT로 통일하는 경우가 많다.

**3. 예시 (RIGHT → LEFT 변환)**
같은 결과를 RIGHT JOIN과 LEFT JOIN 두 방식으로 쓸 수 있다.

```sql
-- RIGHT JOIN: 모든 장르 + 트랙 수 (트랙 없는 장르도 포함)
SELECT g.Name AS GenreName, COUNT(t.TrackId) AS TrackCount
FROM tracks t
RIGHT JOIN genres g ON t.GenreId = g.GenreId
GROUP BY g.GenreId, g.Name
ORDER BY TrackCount DESC;

-- 테이블 순서를 바꾸면 동일한 결과를 LEFT JOIN으로
SELECT g.Name AS GenreName, COUNT(t.TrackId) AS TrackCount
FROM genres g
LEFT JOIN tracks t ON g.GenreId = t.GenreId
GROUP BY g.GenreId, g.Name
ORDER BY TrackCount DESC;
```

**4. 헷갈리기 쉬운 점**
SQLite에서는 `RIGHT JOIN`과 `FULL OUTER JOIN`이 버전 3.39.0(2022년) 이상에서만 지원된다. 구버전이라면 **테이블 순서를 바꿔 LEFT JOIN으로 대체**하거나, FULL은 `LEFT JOIN UNION RIGHT JOIN`으로 흉내 낸다(`UNION`이 중복을 제거해 준다).

**5. 한 줄 정리**
전부 남길 표가 왼쪽이면 LEFT, 오른쪽이면 RIGHT, 양쪽 다면 FULL이다.

> 비유: 단체 사진에서 "우리 반 전원은 무조건 찍고(기준), 옆 반은 짝지어 온 사람만 같이 찍기."

### "없는 데이터" 찾기 — IS NULL 패턴

**1. 정의**
LEFT JOIN으로 모두 남긴 뒤, **짝이 없어 NULL이 된 행만** `IS NULL`로 걸러 내는 기법이다.

**2. 왜 필요한가?**
"주문 안 한 고객", "한 번도 안 팔린 상품", "앨범 없는 아티스트"처럼 **존재하지 않는 관계**를 찾는 질문은 실무에서 매우 흔하다. 이탈 고객 분석, 재고 정리, 데이터 품질 점검에 직결된다.

**3. 예시 (앨범이 한 장도 없는 아티스트)**
```sql
SELECT ar.Name AS ArtistName
FROM artists ar
LEFT JOIN albums al ON ar.ArtistId = al.ArtistId
WHERE al.AlbumId IS NULL
ORDER BY ar.Name;
```

**4. 헷갈리기 쉬운 점**
여기서 `WHERE`를 잘못 쓰면 OUTER JOIN이 무력화된다. 조인 조건은 `ON`에, "짝 없는 행만 남기는" 필터는 `WHERE`에 둬야 한다. 만약 오른쪽 표 컬럼에 일반 조건(`WHERE al.Title = '...'`)을 걸면 NULL 행이 모두 걸러져 사실상 INNER JOIN처럼 변한다.

**5. 한 줄 정리**
"없는 것"을 찾으려면 LEFT JOIN으로 다 남긴 뒤 `오른쪽키 IS NULL`로 짝 없는 행만 추린다.

## 코드로 보기 — 한 번도 안 팔린 트랙 찾기

```sql
SELECT t.Name   AS TrackName,
       al.Title AS AlbumTitle
FROM tracks t
LEFT JOIN invoice_items ii ON t.TrackId = ii.TrackId
LEFT JOIN albums         al ON t.AlbumId = al.AlbumId
WHERE ii.InvoiceLineId IS NULL
ORDER BY al.Title, t.Name;
```

**코드목적**
판매 내역(invoice_items)에 한 번도 등장하지 않은 트랙, 즉 "안 팔린 트랙"의 이름과 소속 앨범을 찾는다.

**해석**
모든 트랙을 기준(왼쪽)으로 두고 판매 내역을 LEFT JOIN하면, 팔린 트랙은 판매 정보가 붙고 안 팔린 트랙은 판매 컬럼이 NULL이 된다. 그 NULL 행만 `WHERE ii.InvoiceLineId IS NULL`로 골라내면 정확히 "안 팔린 트랙"이 남는다. 앨범 제목은 보기 좋게 한 번 더 LEFT JOIN으로 붙였다.

**실무 연결**
"한 번도 주문 안 한 고객", "조회는 됐지만 구매로 이어지지 않은 상품"처럼 **전환되지 않은 대상**을 뽑는 모든 분석이 이 패턴이다. 마케팅 타깃 추출, 재고/콘텐츠 정리의 출발점이다.

## 직접 해보기

1. 모든 고객(customers)과 주문(invoices)을 LEFT JOIN해, 주문이 없는 고객도 포함하여 (고객명·주문ID·날짜·총액)을 조회해 보세요.
2. 각 아티스트의 앨범 수를 LEFT JOIN으로 집계하되, 앨범이 없는 아티스트는 `0`으로 표시되게 해 보세요. (힌트: `COUNT(al.AlbumId)`)
3. artists와 albums를 FULL OUTER JOIN해 보고, SQLite 구버전이라면 `LEFT JOIN UNION RIGHT JOIN`으로 같은 결과를 만들어 보세요.

## 헷갈리기 쉬운 포인트

- **INNER JOIN vs LEFT JOIN**: INNER는 짝 있는 행만, LEFT는 왼쪽 전부 + 짝 없으면 NULL. "전체를 다 보고 싶다"면 LEFT.
- **ON 조건 vs WHERE 조건**: `ON`은 조인 방식, `WHERE`는 조인 후 필터. OUTER JOIN에서 오른쪽 컬럼에 WHERE 조건을 걸면 NULL 행이 사라져 의도가 깨진다.
- **`= NULL` vs `IS NULL`**: NULL은 `=`로 비교 불가. 반드시 `IS NULL` / `IS NOT NULL`.
- **RIGHT JOIN vs LEFT JOIN**: 테이블 순서만 바꾸면 서로 변환된다. 실무에선 LEFT로 통일하는 편.

## 연결되는 개념

- 이전 글: [① JOIN 기초](01-join-basics-inner.md) · [② USING·NATURAL·CROSS](02-using-natural-cross-join.md)
- 다음 글: [④ 셀프 조인](04-self-join.md) — 한 표를 둘로 나눠 잇기
- 함께 보면 좋은: [⑥ 서브쿼리 반환 형태](06-subquery-return-shape.md)의 `NOT IN` / `NOT EXISTS` — "없는 것 찾기"의 또 다른 방법
- 더 찾아볼 키워드: `NULL 처리`, `COALESCE`, `이탈 고객 분석`

## 셀프 체크

- [ ] OUTER JOIN이 짝 없는 행을 NULL로 남긴다는 것을 안다.
- [ ] LEFT/RIGHT/FULL의 "남기는 방향"을 구분할 수 있다.
- [ ] "없는 데이터"를 `LEFT JOIN + IS NULL`로 찾을 수 있다.
- [ ] ON 조건과 WHERE 조건의 차이가 OUTER JOIN에서 결정적임을 안다.
- [ ] OUTER JOIN에서 기준 테이블을 어디에 둘지 판단할 수 있다.

**복습 질문 및 답변**

- (기본) LEFT JOIN에서 오른쪽 표에 짝이 없으면 그 컬럼은 어떤 값이 되나요?
  → NULL이 됩니다.
- (이해 확인) "주문을 한 번도 안 한 고객"을 찾을 때 기준 테이블은 어느 쪽이어야 하나요?
  → 전체를 빠짐없이 봐야 하는 `customers`를 기준(왼쪽)에 두고, `orders`를 LEFT JOIN한 뒤 주문 쪽 키가 `IS NULL`인 행을 고릅니다.
- (응용) OUTER JOIN 후 오른쪽 표 컬럼에 `WHERE`로 일반 조건을 걸면 왜 위험한가요?
  → NULL 행이 조건을 통과하지 못해 모두 걸러지면서, OUTER JOIN이 사실상 INNER JOIN처럼 동작하기 때문입니다.

## 한 줄 정리

> OUTER JOIN은 짝 없는 행까지 NULL로 남기는 조인이며, "없는 데이터"는 LEFT JOIN으로 다 남긴 뒤 `IS NULL`로 찾는다.
