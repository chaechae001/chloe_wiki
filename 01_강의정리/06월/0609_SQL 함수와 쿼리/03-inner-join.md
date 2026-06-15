# INNER JOIN — 나뉜 두 테이블을 키로 이어 붙이기

> 대여 기록에는 user_id만 있고 이름은 없다. 회원 이름과 대여 내역을 한 화면에서 보려면, 흩어진 두 표를 어떻게 합칠까?

`INNER JOIN` `ON` `외래키` `교집합` `테이블연결` `ambiguous` `별칭`

## 핵심요약

- 데이터는 보통 여러 테이블로 **나뉘어** 저장된다(중복 방지·관리 편의·무결성 때문).
- `JOIN`은 나뉜 테이블을 **공통 키**로 다시 이어 붙여 한 번에 조회하는 명령이다.
- `INNER JOIN`은 양쪽에 **매칭되는 키가 있는 행만**(교집합) 가져온다.
- 연결 조건은 `ON 테이블A.키 = 테이블B.키`로 명시한다.
- 두 테이블에 **같은 이름의 컬럼**이 있으면 `테이블명.컬럼명`으로 구분해야 모호성 오류가 안 난다.
- 실무 팁: 먼저 `SELECT *`로 조인 성공 여부를 확인한 뒤, 필요한 컬럼만 골라 좁힌다.

## 개념별 정리

### 왜 테이블을 나눠 저장할까?

**1. 정의**
하나의 큰 표에 모든 정보를 담지 않고, 주제별로 여러 테이블로 쪼개어 저장하는 설계 방식이다(정규화).

**2. 왜 필요한가?**
"회원 정보 + 대여 기록을 한 표에 다 넣으면 안 되나?"라는 의문이 자연스럽다. 하지만 한 표에 다 넣으면 문제가 생긴다. 같은 회원이 100번 빌리면 회원 이름·주소가 100번 중복 저장되고(저장 낭비), 그 회원이 이사하면 100줄을 전부 고쳐야 하며(수정 부담), 한 줄이라도 빠뜨리면 데이터가 어긋난다(무결성 깨짐). 그래서 회원은 `user` 테이블, 대여는 `rental` 테이블로 나누고, 둘을 `user_id` 같은 키로 연결한다.

**3. 예시**
`user` 테이블에는 회원의 이름·연락처가, `rental` 테이블에는 누가(`user_id`) 무엇을 빌렸는지가 들어 있다. 두 테이블은 `user.id = rental.user_id`로 이어진다.

**4. 헷갈리기 쉬운 점**
나누는 것이 비효율처럼 보이지만, 오히려 중복과 수정 오류를 줄여 준다. JOIN은 "나뉜 것을 합치는 비용"이 아니라 "나눠 둔 덕분에 깔끔하게 합칠 수 있는 도구"다.

**5. 한 줄 정리**
테이블을 나누는 이유는 중복·수정·무결성 문제를 피하기 위함이고, JOIN은 그것을 다시 합치는 방법이다.

### INNER JOIN — 교집합만 가져오기

**1. 정의**
두 테이블에서 연결 키가 서로 **일치하는 행만** 골라 하나의 결과로 합치는 조인이다.

**2. 왜 필요한가?**
"실제로 대여 기록이 있는 회원의 이름과 그 대여 내역"처럼, 양쪽에 모두 존재하는 데이터만 보고 싶을 때 쓴다.

**3. 예시**

```sql
SELECT *
FROM rental
INNER JOIN user
ON user.id = rental.user_id;
```

`rental`과 `user`를 `user.id = rental.user_id` 조건으로 연결한다. `rental`의 `user_id`와 일치하는 `user`가 있는 행만 결과에 남는다. 대여 기록이 없는 회원이나, 회원 정보가 없는 대여 기록은 빠진다(그래서 "교집합").

**4. 헷갈리기 쉬운 점**
`ON`을 빠뜨리고 그냥 두 테이블을 붙이면, 모든 행이 서로 곱해진 의미 없는 결과가 나올 수 있다. 어떤 키로 연결할지(`ON`)를 반드시 명시해야 한다. 연결 키는 보통 ERD(테이블 관계도)를 보고 확인한다.

**5. 한 줄 정리**
INNER JOIN은 "양쪽에 다 있는 것만" 골라 합치는, 교집합 방식의 연결이다.

> 비유: 두 개의 명단(참석 신청자 / 실제 결제자)을 겹쳐 놓고, 양쪽에 모두 이름이 있는 사람만 추려내는 것과 같다.

### table.column — 같은 이름 컬럼의 모호성 해결

**1. 정의**
어느 테이블의 컬럼인지 `테이블명.컬럼명` 형태로 명시하는 표기법이다.

**2. 왜 필요한가?**
조인하면 두 테이블의 컬럼이 한 결과에 모인다. 이때 `track`과 `genre`에 둘 다 `name` 컬럼이 있으면, 그냥 `name`이라고 쓰면 DBMS가 "어느 쪽 name이냐"며 혼동해 `ambiguous column name`(모호한 컬럼명) 오류를 낸다.

**3. 예시**

```sql
SELECT tracks.name, genres.name
FROM tracks
INNER JOIN genres
ON tracks.genre_id = genres.genre_id;
```

`tracks.name`(곡 이름)과 `genres.name`(장르 이름)을 명시해 모호성을 없앤다.

**4. 헷갈리기 쉬운 점**
컬럼 이름이 겹치지 않으면 `테이블명.`을 생략해도 되지만, 조인이 많아질수록 명시하는 습관이 가독성과 안정성을 높인다. 별칭(alias)을 붙여 `t.name`, `g.name`처럼 짧게 쓸 수도 있는데, 별칭 동작은 DBMS마다 미세하게 다를 수 있어 처음에는 테이블명을 그대로 명시하는 편이 헷갈림이 적다.

**5. 한 줄 정리**
컬럼 이름이 겹칠 땐 `테이블명.컬럼명`으로 어느 테이블 것인지 분명히 한다.

## 코드로 보기 — 단계적으로 조인 좁히기

조인은 한 번에 완성하려 하지 말고 두 단계로 접근하면 실수가 줄어든다.

```sql
-- 1단계: 조인 자체가 성공하는지 SELECT * 로 먼저 확인
SELECT *
FROM albums
INNER JOIN artists
ON albums.artist_id = artists.artist_id;

-- 2단계: 성공을 확인한 뒤 필요한 컬럼만 좁히기
SELECT albums.title, artists.name
FROM albums
INNER JOIN artists
ON albums.artist_id = artists.artist_id;
```

**코드목적**
앨범(`albums`)과 아티스트(`artists`)를 `artist_id`로 연결해, "앨범 제목 옆에 그 앨범을 만든 아티스트 이름"이 함께 보이도록 하는 것이 목적이다.

**해석**
1단계에서 `SELECT *`는 두 테이블의 모든 컬럼을 그대로 붙여 보여 주므로, 연결이 제대로 됐는지(엉뚱하게 곱해지지 않았는지) 눈으로 확인하기 좋다. 연결이 맞다는 걸 확인한 뒤, 2단계에서 `albums.title`과 `artists.name`만 골라 깔끔한 결과로 좁힌다.

**실무 연결**
주문 테이블의 `customer_id`를 고객 테이블의 이름과 연결해 "주문번호·금액·고객명"을 함께 보는 보고서, 상품 코드와 상품명을 잇는 매출표 등 거의 모든 실무 조회가 이 패턴이다. 다만 실무 최종 쿼리에서는 `SELECT *`를 피하고 필요한 컬럼만 명시한다(성능·가독성 때문). `SELECT *`는 어디까지나 "조인 확인용"으로만 쓴다.

## 직접 해보기

1. `tracks`와 `genres`를 `genre_id`로 조인해, 곡 이름과 장르 이름을 함께 출력해 보자. (컬럼명 `name`이 겹치는 점에 주의)
2. `invoice`와 `customer`를 `customer_id`로 조인해, `invoice_id`, `total`, `first_name`, `last_name`을 출력해 보자.
3. `invoice_items`와 `tracks`를 `track_id`로 조인해 보자. 먼저 `SELECT *`로 성공을 확인한 뒤 필요한 컬럼만 좁혀 보자.

<details>
<summary>예시 답안 보기</summary>

```sql
-- 1번
SELECT tracks.name, genres.name
FROM tracks
INNER JOIN genres
ON tracks.genre_id = genres.genre_id;

-- 2번
SELECT invoice.invoice_id, invoice.total, customer.first_name, customer.last_name
FROM invoice
INNER JOIN customer
ON invoice.customer_id = customer.customer_id;

-- 3번 (확인 후 좁히기)
SELECT *
FROM invoice_items
INNER JOIN tracks
ON invoice_items.track_id = tracks.track_id;
```

</details>

## 헷갈리기 쉬운 포인트

- **ON을 빠뜨림 vs 명시**: 연결 조건(`ON`) 없이 조인하면 의미 없는 거대한 결과가 나올 수 있다. 항상 어떤 키로 잇는지 명시한다.
- **name vs tracks.name**: 컬럼명이 겹치면 `테이블명.컬럼명`으로 구분한다. 안 그러면 모호성 오류가 난다.
- **SELECT \* (확인용) vs 컬럼 명시(실무용)**: 조인 확인엔 `*`가 편하지만, 최종 쿼리는 필요한 컬럼만 골라 쓴다.

## 연결되는 개념

- 이전 글: [GROUP BY와 HAVING](02-group-by-having.md) — 그룹별 집계로 나온 id를 이름과 연결할 때 JOIN이 이어진다.
- 다음 글: [LEFT / RIGHT JOIN과 NULL](04-outer-join-and-null.md) — "교집합만"으로는 빠지는 데이터(대여 없는 회원 등)를 포함하는 방법.
- 더 찾아볼 키워드: 외래키(FK)·기본키(PK), ERD(테이블 관계도), 3개 이상 테이블 조인, 별칭(alias)

## 셀프 체크

- [ ] 테이블을 나눠 저장하는 이유(중복·수정·무결성)를 설명할 수 있다.
- [ ] INNER JOIN이 교집합만 가져온다는 것을 안다.
- [ ] `ON`으로 연결 키를 지정하는 문법을 쓸 수 있다.
- [ ] 컬럼명이 겹칠 때 `테이블명.컬럼명`으로 구분할 수 있다.
- [ ] "SELECT \*로 확인 후 컬럼 좁히기" 흐름을 따라 할 수 있다.

**복습 질문 및 답변**

*기본* — 두 테이블을 연결하는 조건은 어떤 절로 지정하나?
> `ON` 절로 지정한다. 예: `ON user.id = rental.user_id`.

*이해 확인* — INNER JOIN 결과에서 "대여 기록이 없는 회원"은 나올까?
> 나오지 않는다. INNER JOIN은 양쪽에 매칭되는 키가 있는 행(교집합)만 가져오기 때문이다.

*응용* — `tracks`와 `genres`에 둘 다 `name`이 있을 때 `SELECT name ...`이라고 쓰면 무슨 일이 일어나나?
> 어느 테이블의 `name`인지 모호해 `ambiguous column name` 오류가 난다. `tracks.name`, `genres.name`처럼 명시해야 한다.

## 한 줄 정리

> INNER JOIN은 나뉜 두 테이블을 공통 키로 이어 붙여, 양쪽에 모두 존재하는 행만(교집합) 한 화면에 보여 준다. 연결 조건은 `ON`으로 명시하고, 컬럼명이 겹치면 `테이블명.컬럼명`으로 구분한다.
