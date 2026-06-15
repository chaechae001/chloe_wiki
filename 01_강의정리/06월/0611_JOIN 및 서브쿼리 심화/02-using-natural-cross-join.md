# USING · NATURAL · CROSS JOIN — 같은 컬럼으로 잇기, 모든 조합 만들기

> ON절로 매번 `테이블1.컬럼 = 테이블2.컬럼`을 쓰기 번거로울 때, 더 짧게 잇는 문법이 있습니다. 반대로 "아예 모든 조합"을 만들고 싶을 때도요.

`USING` `NATURAL JOIN` `CROSS JOIN` `카티션곱` `별칭제약` `등가조인`

## 핵심요약

- **USING**: 두 표에 **이름이 같은 컬럼**이 있을 때, 그 컬럼만 지정해 간단히 등가 조인한다.
- **NATURAL JOIN**: 이름이 같은 **모든** 컬럼을 자동으로 등가 조인 조건으로 삼는다.
- USING·NATURAL JOIN은 **별칭(alias)을 붙일 수 없다**는 제약이 있다.
- **CROSS JOIN**: 조인 조건 없이 **모든 행의 조합**을 만든다(카티션 곱). 결과 행 수 = A 행 수 × B 행 수.
- CROSS JOIN은 결과가 급격히 커지므로 실무에서는 특수한 경우에만 쓴다.
- 편의 문법은 편하지만 제약이 많아, 강의에서도 **개념만 경험**하고 평소엔 `JOIN ... ON ...`을 권장한다.

## 개념별 정리

### USING 조건절

**1. 정의**
두 표에 **같은 이름을 가진 컬럼**이 있을 때, 그 컬럼을 기준으로 등가 조인하는 간편 문법이다.

**2. 왜 필요한가?**
연결 컬럼 이름이 양쪽 다 `ArtistId`로 동일하다면, `ON a.ArtistId = b.ArtistId`라고 길게 쓰는 대신 `USING (ArtistId)`로 짧게 끝낼 수 있다.

**3. 예시**
```sql
SELECT albums.Title  AS AlbumTitle,
       artists.Name  AS ArtistName
FROM albums
INNER JOIN artists USING (ArtistId)
ORDER BY albums.Title
LIMIT 10;
```

**4. 헷갈리기 쉬운 점**
USING을 쓰면 **컬럼이나 테이블에 별칭을 붙일 수 없다**. 또 일부 환경(SQL Server)에서는 USING 자체를 지원하지 않는다. 연결 컬럼 이름이 같을 때만 쓸 수 있다는 전제도 기억해야 한다.

**5. 한 줄 정리**
이름이 같은 컬럼 하나로 간단히 잇고 싶을 때 쓰는 축약 문법이다.

### NATURAL JOIN

**1. 정의**
두 표에서 **이름이 같은 모든 컬럼**을 자동으로 찾아 등가 조인 조건으로 사용하는 조인이다.

**2. 왜 필요한가?**
조인 조건을 한 글자도 쓰지 않아도 알아서 같은 이름 컬럼을 맞춰 준다. 문법이 가장 짧다.

**3. 예시**
```sql
SELECT albums.Title  AS AlbumTitle,
       artists.Name  AS ArtistName
FROM albums
NATURAL JOIN artists
ORDER BY albums.Title
LIMIT 10;
```

**4. 헷갈리기 쉬운 점**
"자동"이라 편해 보이지만 위험하다. 이름이 같은 컬럼이 **여러 개**면 그 전부가 조인 조건이 되어 의도와 다른 결과가 나온다. 예를 들어 트랙과 장르는 `GenreId` 말고 `Name`도 공통이라, NATURAL JOIN하면 "트랙명 = 장르명"까지 조건이 되어 결과가 0행이 된다. 그래서 **공통 컬럼이 하나뿐인 표 쌍**에서만 안전하다. 별칭도 붙일 수 없고 ON·USING·WHERE로 조인 조건을 추가할 수도 없다.

**5. 한 줄 정리**
같은 이름 컬럼을 전부 조건으로 삼는 자동 조인 — 편하지만 통제력이 약하다.

> 비유: 이름이 같으면 무조건 짝으로 묶는 자동 매칭. 동명이인이 많으면 엉뚱하게 묶인다.

### CROSS JOIN

**1. 정의**
조인 조건 없이 한쪽 표의 모든 행을 다른 쪽 표의 모든 행과 **하나씩 전부 조합**하는 조인이다. 수학의 카티션 곱(곱집합)에 해당한다.

**2. 왜 필요한가?**
"가능한 모든 경우의 수"를 만들어야 할 때 쓴다. 예를 들어 사람 2명과 교통수단 3가지의 모든 이동 조합(2×3=6가지)을 만들거나, 달력의 날짜 × 매장 조합처럼 빈칸까지 포함한 기준표를 만들 때다.

**3. 예시**
```sql
SELECT g.Name  AS GenreName,
       m.Name  AS MediaTypeName
FROM genres g
CROSS JOIN media_types m
ORDER BY g.Name, m.Name;
```

**4. 헷갈리기 쉬운 점**
결과 행 수가 **곱셈으로 폭증**한다. 100행 × 100행이면 10,000행이다. 조인 조건을 깜빡 잊으면 의도치 않게 CROSS JOIN이 되어 결과가 터지는 사고가 흔하다. 행 수를 미리 가늠하고 써야 한다.

**5. 한 줄 정리**
CROSS JOIN은 조건 없이 모든 조합을 만드는 조인이며, 결과 크기에 항상 주의해야 한다.

> 비유: 메뉴판의 "음료 3종 × 사이즈 2종 = 6가지 주문 조합"을 빠짐없이 적는 것.

## 코드로 보기 — CROSS JOIN으로 행 수 확인하기

```sql
-- 모든 조합 만들기
SELECT g.Name AS GenreName, m.Name AS MediaTypeName
FROM genres g
CROSS JOIN media_types m;

-- 그 결과가 몇 행인지 검산
SELECT COUNT(*) AS TotalRows
FROM genres
CROSS JOIN media_types;
```

**코드목적**
장르와 미디어타입의 모든 조합을 만들고, 그 행 수가 정말 `장르 수 × 미디어타입 수`인지 직접 확인한다.

**해석**
첫 쿼리는 조합 자체를, 둘째 쿼리는 그 개수를 센다. `COUNT(*)` 결과는 `(장르 개수) × (미디어타입 개수)`와 정확히 일치해야 한다. 이렇게 검산해 보면 CROSS JOIN이 왜 "곱"인지 몸으로 이해된다.

**실무 연결**
"모든 날짜 × 모든 지점"처럼 비어 있는 칸까지 포함한 완전한 격자가 필요할 때(예: 판매가 0인 날도 그래프에 표시) CROSS JOIN으로 뼈대를 만든 뒤 실제 데이터를 LEFT JOIN으로 붙인다.

## 직접 해보기

1. 플레이리스트(`PlaylistId` 1~5)와 장르(`GenreId` 1~3)의 모든 조합을 CROSS JOIN으로 만들어 보세요. (총 15행 예상)
2. 앨범과 아티스트를 `USING (ArtistId)`로 조인해 상위 10행을 조회하고, 같은 결과를 `ON al.ArtistId = ar.ArtistId`로도 작성해 비교해 보세요.
3. 트랙과 장르를 NATURAL JOIN하면 왜 결과가 비는지 설명해 보고, 안전하게 INNER JOIN(ON절)으로 바꿔 보세요.

## 헷갈리기 쉬운 포인트

- **USING vs ON**: USING은 "이름이 같은 컬럼"에만 쓰는 축약형이고 별칭 금지. ON은 어떤 컬럼이든 자유롭게 조건 지정 가능. 평소엔 ON이 안전하다.
- **NATURAL JOIN vs USING**: NATURAL은 같은 이름 컬럼을 **전부 자동** 사용, USING은 **내가 고른** 컬럼만 사용. NATURAL이 더 위험하다.
- **CROSS JOIN vs INNER JOIN**: CROSS는 조건이 없어 모든 조합, INNER는 조건이 맞는 조합만. 조인 조건을 빠뜨린 INNER JOIN은 사실상 CROSS JOIN이 된다.

## 연결되는 개념

- 이전 글: [① JOIN 기초](01-join-basics-inner.md) — INNER JOIN과 ON절
- 다음 글: [③ OUTER JOIN](03-outer-join.md) — 짝 없는 행도 남기기
- 더 찾아볼 키워드: `카티션 곱(Cartesian product)`, `곱집합`, `데이터 격자(grid) 만들기`

## 셀프 체크

- [ ] USING이 "이름이 같은 컬럼"에만 쓰는 축약형임을 안다.
- [ ] NATURAL JOIN이 같은 이름 컬럼을 전부 조건으로 삼는다는 점과 그 위험을 안다.
- [ ] USING·NATURAL JOIN에 별칭을 못 붙인다는 제약을 안다.
- [ ] CROSS JOIN의 결과 행 수가 A×B임을 계산할 수 있다.
- [ ] 조인 조건을 빠뜨리면 의도치 않게 CROSS JOIN이 됨을 안다.

**복습 질문 및 답변**

- (기본) CROSS JOIN의 결과 행 수는 어떻게 계산하나요?
  → 두 표의 행 수를 곱합니다(A 행 수 × B 행 수).
- (이해 확인) 트랙과 장르를 NATURAL JOIN하면 왜 결과가 0행이 되나요?
  → 두 표의 공통 컬럼이 `GenreId`와 `Name` 두 개라, "트랙명 = 장르명"까지 조건이 되어 일치하는 행이 없기 때문입니다.
- (응용) "모든 날짜 × 모든 지점" 격자를 만들고 실제 매출을 채우려면 어떤 조인 조합을 쓰나요?
  → CROSS JOIN으로 빈칸 없는 격자를 만든 뒤, 실제 매출 데이터를 LEFT JOIN으로 붙입니다.

## 한 줄 정리

> USING·NATURAL은 같은 이름 컬럼으로 잇는 축약 문법(제약 많음)이고, CROSS JOIN은 조건 없이 모든 조합을 만드는 조인(결과 크기 주의)이다.
