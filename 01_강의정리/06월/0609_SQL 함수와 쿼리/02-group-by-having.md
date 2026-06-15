# GROUP BY와 HAVING — 그룹으로 묶어 요약하고, 그룹에 조건을 걸기

> 회원별로 책을 몇 번 빌렸는지, 나라별 매출이 얼마인지. "전체 한 줄 요약"이 아니라 "그룹마다 따로 요약"하려면 어떻게 할까?

`GROUP BY` `HAVING` `집계함수` `그룹화` `WHERE와의차이` `매출집계`

## 핵심요약

- `GROUP BY`는 **같은 값을 가진 행끼리 묶어**, 그룹마다 집계 함수를 적용한다.
- 집계 함수(`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`)와 거의 항상 짝을 이룬다.
- 원칙적으로 **GROUP BY에 쓴 컬럼은 SELECT에도 함께 써야** 결과의 의미가 분명해진다.
- `HAVING`은 **그룹으로 묶은 뒤** 그 결과에 조건을 거는 절이다.
- `WHERE`는 묶기 전 개별 행에 거는 조건, `HAVING`은 묶은 후 그룹에 거는 조건이다.
- `HAVING`은 단독으로 못 쓰며 반드시 `GROUP BY` 다음에 온다.

## 개념별 정리

### GROUP BY — 같은 값끼리 묶기

**1. 정의**
지정한 컬럼의 값이 같은 행들을 하나의 그룹으로 묶고, 그룹마다 집계 함수를 한 번씩 적용하는 절이다.

**2. 왜 필요한가?**
앞 글에서 `COUNT(*)`는 테이블 전체를 한 숫자로 요약했다. 하지만 실무에서는 "전체 회원 수"보다 "회원 한 명 한 명이 몇 번 대여했는지"가 더 궁금할 때가 많다. GROUP BY는 이 "기준별 요약"을 가능하게 한다.

**3. 예시**

```sql
SELECT user_id, COUNT(*)
FROM rental
GROUP BY user_id;
```

`rental`(대여 기록) 테이블을 `user_id`가 같은 것끼리 묶고, 그룹마다 행이 몇 개인지(=몇 번 빌렸는지) 센다.

집계 함수를 바꾸면 묻는 내용이 달라진다.

```sql
-- 회원별 대여한 금액 합계
SELECT user_id, SUM(price) FROM rental GROUP BY user_id;

-- 회원별 대여 금액 평균
SELECT user_id, AVG(price) FROM rental GROUP BY user_id;

-- 회원별 최고/최저 금액
SELECT user_id, MAX(price) FROM rental GROUP BY user_id;
SELECT user_id, MIN(price) FROM rental GROUP BY user_id;
```

**4. 헷갈리기 쉬운 점**
GROUP BY에 쓴 컬럼은 SELECT에도 넣는 것이 원칙이다. 그래야 "이 숫자가 어느 그룹의 것인지" 알 수 있다. 일부 DBMS(예: SQLite)는 SELECT에 그룹 기준이 아닌 컬럼을 넣어도 오류 없이 결과를 내기도 하지만, 그렇게 나온 값은 "그 그룹의 어떤 한 행"이 임의로 뽑힌 것이라 의미가 모호하다. 결과가 나오더라도 정석 문법을 지키는 편이 안전하다.

**5. 한 줄 정리**
GROUP BY는 "기준 컬럼이 같은 행끼리 묶어 그룹마다 한 줄로 요약"하는 절이다.

> 비유: 학생 명단을 반별로 모아 책상마다 쌓아 두고, 각 책상 위 종이 수를 세는 것과 같다. 책상(그룹)마다 결과가 하나씩 나온다.

### HAVING — 그룹에 조건 걸기

**1. 정의**
GROUP BY로 묶은 결과에 대해 추가로 조건을 거는 절이다.

**2. 왜 필요한가?**
"회원별 대여 횟수"를 구했더니 1번만 빌린 사람이 너무 많다고 하자. "2번 이상 빌린 사람만" 보고 싶을 때, 그 조건은 개별 행이 아니라 **그룹의 집계 결과**(대여 횟수)에 걸어야 한다. 이때 쓰는 것이 HAVING이다.

**3. 예시**

```sql
SELECT user_id, COUNT(*)
FROM rental
GROUP BY user_id
HAVING COUNT(user_id) > 1;
```

회원별로 묶은 뒤, 대여 건수가 1을 초과(=2건 이상)하는 회원만 남긴다.

**4. 헷갈리기 쉬운 점**
HAVING은 `GROUP BY` 없이 단독으로 쓰지 않는다. 항상 그룹화가 먼저고, 그 결과를 거르는 것이 HAVING이다. 또한 `WHERE`와 역할이 다르다(아래 비교 참고).

**5. 한 줄 정리**
HAVING은 "묶은 다음, 그룹의 집계 결과를 기준으로 거르는" 조건이다.

## 코드로 보기 — 나라별 매출 집계 후 정렬

```sql
-- billing_country별 매출 합계를 구하고, 큰 순서로 정렬
SELECT billing_country, SUM(total) AS revenue
FROM invoice
GROUP BY billing_country
ORDER BY revenue DESC;
```

**코드목적**
주문 내역(`invoice`)을 나라별로 묶어 매출 합계를 구하고, 매출이 큰 나라부터 보이도록 정렬하는 것이 목적이다.

**해석**
`GROUP BY billing_country`로 같은 나라의 주문이 한 줄로 합쳐지고, `SUM(total)`이 그 나라의 총매출이 된다. 마지막 `ORDER BY revenue DESC`는 합계가 큰 나라를 맨 위로 올린다. 즉 "어느 나라에서 가장 많이 팔렸나"가 한눈에 보인다.

여기에 조건을 더하고 싶다면, "묶기 전"이냐 "묶은 후"냐로 절을 고른다.

```sql
-- 묶기 전: 특정 기간 주문만 대상으로 (WHERE)
SELECT billing_country, SUM(total) AS revenue
FROM invoice
WHERE total > 0
GROUP BY billing_country
HAVING SUM(total) >= 100      -- 묶은 후: 매출 100 이상 나라만
ORDER BY revenue DESC;
```

**실무 연결**
"지역별 매출", "상품군별 판매량", "요일별 방문자 수"처럼 보고서에 자주 등장하는 표는 대부분 `GROUP BY` + 집계 + `ORDER BY`의 조합이다. 여기에 `HAVING`을 더하면 "기준 미달 그룹 제외" 같은 필터까지 한 번에 처리된다.

## 직접 해보기

1. `album` 테이블에서 `artist_id`별로 앨범이 몇 장인지 집계해 보자.
2. `customer` 테이블에서 `country`별 고객 수를 구하고, 고객이 많은 나라부터 정렬해 보자.
3. 위 2번 결과에서 "고객이 5명 이상인 나라만" 남기려면 `WHERE`와 `HAVING` 중 무엇을 써야 할까? 직접 쿼리를 완성해 보자.

<details>
<summary>예시 답안 보기</summary>

```sql
-- 1번
SELECT artist_id, COUNT(*) FROM album GROUP BY artist_id;

-- 2번
SELECT country, COUNT(customer_id) AS cnt
FROM customer
GROUP BY country
ORDER BY cnt DESC;

-- 3번: "고객 수"는 그룹의 집계 결과이므로 HAVING을 쓴다.
SELECT country, COUNT(customer_id) AS cnt
FROM customer
GROUP BY country
HAVING COUNT(customer_id) >= 5
ORDER BY cnt DESC;
```

</details>

## 헷갈리기 쉬운 포인트

- **WHERE vs HAVING**: `WHERE`는 그룹으로 묶기 **전** 개별 행을 거른다. `HAVING`은 묶은 **후** 그룹의 집계 결과를 거른다. "각 행의 가격이 1000원 이상"은 WHERE, "그룹 합계가 1만 원 이상"은 HAVING이다.
- **GROUP BY 컬럼 = SELECT 컬럼**: 그룹 기준은 SELECT에도 넣어야 결과 해석이 분명하다.
- **HAVING 단독 사용 불가**: HAVING은 항상 GROUP BY 뒤에 온다.

## 연결되는 개념

- 이전 글: [집계 함수와 LIMIT](01-aggregate-functions.md) — GROUP BY는 집계 함수를 "그룹별"로 쓰는 것이므로 먼저 익혀 두면 좋다.
- 다음 글: [INNER JOIN](03-inner-join.md) — 그룹별 집계로 나온 id를 실제 이름·정보와 연결하려면 JOIN이 필요하다.
- 더 찾아볼 키워드: `ORDER BY`(정렬), `DISTINCT`(중복 제거), SQL 실행 순서(WHERE → GROUP BY → HAVING → SELECT → ORDER BY)

## 셀프 체크

- [ ] GROUP BY가 "행을 그룹으로 묶는" 절임을 설명할 수 있다.
- [ ] GROUP BY 컬럼을 SELECT에도 넣어야 하는 이유를 안다.
- [ ] HAVING과 WHERE의 차이를 예로 들 수 있다.
- [ ] HAVING이 GROUP BY 없이는 쓰이지 않음을 안다.
- [ ] GROUP BY + 집계 + ORDER BY로 "그룹별 순위표"를 만들 수 있다.

**복습 질문 및 답변**

*기본* — 회원별 대여 횟수를 구하려면 어떤 절을 써야 하나?
> `GROUP BY user_id`로 묶고 `COUNT(*)`로 센다.

*이해 확인* — "각 주문 금액이 0보다 큰 것만"과 "나라별 매출 합계가 100 이상인 것만"은 각각 WHERE/HAVING 중 무엇으로 거르나?
> 앞은 개별 행 조건이므로 WHERE, 뒤는 그룹의 집계 결과 조건이므로 HAVING.

*응용* — 평균 재생시간(`milliseconds`)을 장르별로 구하되 분 단위로 보고 싶다면?
> `AVG(milliseconds) / 60000`처럼 60000으로 나눠 분으로 환산하면 된다. `SELECT genre_id, AVG(milliseconds)/60000 FROM track GROUP BY genre_id;`

## 한 줄 정리

> GROUP BY는 같은 값끼리 묶어 그룹마다 집계하고, HAVING은 그 그룹 결과에 조건을 건다. "묶기 전 조건은 WHERE, 묶은 후 조건은 HAVING"이라는 한 가지만 기억하면 절반은 끝난다.
