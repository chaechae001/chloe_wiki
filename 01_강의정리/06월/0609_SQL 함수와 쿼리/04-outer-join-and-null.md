# LEFT / RIGHT JOIN과 NULL — 한쪽 전체를 기준으로 연결하고, 빈 값을 골라내기

> INNER JOIN은 "양쪽에 다 있는 것"만 보여 준다. 그런데 "한 번도 대여하지 않은 회원"이나 "앨범이 없는 아티스트"는 어떻게 찾을까?

`LEFT JOIN` `RIGHT JOIN` `OUTER JOIN` `IS NULL` `IS NOT NULL` `NULL` `데이터정의서`

## 핵심요약

- `INNER JOIN`은 교집합만 주므로, "한쪽에만 있는 데이터"는 결과에서 사라진다.
- `LEFT JOIN`은 **왼쪽 테이블의 모든 행**을 남기고, 매칭되는 오른쪽 값이 없으면 `NULL`로 채운다.
- `RIGHT JOIN`은 그 반대로 **오른쪽 테이블의 모든 행**을 기준으로 한다.
- 실무에서는 보통 `LEFT JOIN`을 쓰고, 필요하면 두 테이블의 순서를 바꿔 RIGHT 효과를 낸다.
- `IS NULL`은 빈 값인 행만, `IS NOT NULL`은 값이 있는 행만 골라낸다.
- "매칭 안 된 행"을 찾는 핵심 기술이 `LEFT JOIN` + `IS NULL` 조합이다.

## 개념별 정리

### LEFT JOIN — 왼쪽 전체를 기준으로

**1. 정의**
왼쪽(`FROM`에 먼저 쓴) 테이블의 모든 행을 남기고, 오른쪽 테이블에서 매칭되는 값을 붙이는 조인이다. 매칭이 없으면 오른쪽 컬럼은 `NULL`이 된다.

**2. 왜 필요한가?**
INNER JOIN은 "대여 기록이 있는 회원"만 보여 준다. 하지만 "대여를 한 번도 안 한 회원도 포함해 전체 회원을 보고 싶을 때"가 있다. 이때 회원 테이블을 왼쪽에 두고 LEFT JOIN하면, 대여 기록이 없는 회원도 빠지지 않고 남는다(대여 컬럼은 NULL로).

**3. 예시**

```sql
SELECT *
FROM user
LEFT JOIN rental
ON user.id = rental.user_id;
```

`user`(왼쪽)의 모든 회원이 결과에 남는다. 대여 기록이 있는 회원은 `rental` 정보가 붙고, 없는 회원은 `rental` 쪽 컬럼이 NULL로 표시된다.

**4. 헷갈리기 쉬운 점**
"왼쪽"이 어디인지 헷갈린다. `FROM A LEFT JOIN B`에서 왼쪽은 `A`(FROM 뒤의 테이블)다. 기준으로 삼아 "전부 남기고 싶은" 테이블을 왼쪽에 둔다.

**5. 한 줄 정리**
LEFT JOIN은 "왼쪽 테이블은 한 명도 빠뜨리지 않고, 짝이 없으면 빈칸(NULL)으로" 연결한다.

> 비유: 전체 회원 명단(왼쪽)을 그대로 두고, 옆 칸에 대여 기록을 채워 넣되 기록이 없으면 빈칸으로 남기는 것과 같다.

### INNER JOIN vs LEFT JOIN

| 구분 | INNER JOIN | LEFT JOIN |
| --- | --- | --- |
| 남는 행 | 양쪽에 모두 있는 행(교집합) | 왼쪽 테이블의 모든 행 |
| 매칭 없을 때 | 그 행은 사라짐 | 왼쪽은 남고 오른쪽은 NULL |
| 쓰는 상황 | "양쪽에 다 있는 것만" | "왼쪽 전체 + 있으면 붙이기" |

### RIGHT JOIN — 오른쪽 전체를 기준으로

**1. 정의**
LEFT JOIN의 반대로, 오른쪽 테이블의 모든 행을 남기는 조인이다.

**2. 왜 필요한가?**
"회원이 탈퇴해 회원 정보는 없지만 대여 기록은 남아 있는" 경우처럼, 오른쪽 테이블을 기준으로 전부 보고 싶을 때 쓴다.

**3. 예시**

```sql
SELECT *
FROM user
RIGHT JOIN rental
ON user.id = rental.user_id;
```

`rental`(오른쪽)의 모든 대여 기록이 남고, 매칭되는 회원이 없으면 `user` 쪽 컬럼이 NULL이 된다.

**4. 헷갈리기 쉬운 점**
초보 단계에서는 LEFT와 RIGHT가 자주 헷갈린다. 그래서 실무 팁은 **"RIGHT JOIN 대신 LEFT JOIN을 쓰라"** 이다. 두 테이블만 조인하는 경우라면, 테이블 순서를 바꾸면 RIGHT가 필요했던 결과를 LEFT로 똑같이 얻을 수 있다(`A RIGHT JOIN B` = `B LEFT JOIN A`). 다만 테이블이 셋 이상으로 늘어나면 RIGHT가 유용한 상황이 생길 수 있어, 개념 자체는 알아 두는 편이 좋다.

**5. 한 줄 정리**
RIGHT JOIN은 오른쪽 전체를 기준으로 하지만, 보통은 순서를 바꿔 LEFT JOIN으로 대체한다.

### IS NULL / IS NOT NULL — 빈 값 골라내기

**1. 정의**
`IS NULL`은 값이 비어 있는(NULL) 행을, `IS NOT NULL`은 값이 채워진 행을 골라내는 조건이다.

**2. 왜 필요한가?**
LEFT JOIN을 하면 "매칭되지 않은 행"이 NULL로 표시된다. 바로 이 NULL만 골라내면 **"짝이 없는 데이터"** 를 정확히 찾을 수 있다. "앨범이 없는 아티스트", "주문이 없는 고객"이 이렇게 구해진다.

**3. 예시**

```sql
-- 앨범이 없는 아티스트만 찾기
SELECT artists.name
FROM artists
LEFT JOIN albums
ON artists.artist_id = albums.artist_id
WHERE albums.album_id IS NULL;
```

아티스트 전체를 왼쪽에 두고 앨범을 LEFT JOIN하면, 앨범이 없는 아티스트는 `albums` 쪽 컬럼이 NULL이 된다. 그 NULL인 행만 `WHERE albums.album_id IS NULL`로 골라내면 "앨범이 없는 아티스트"가 남는다.

**4. 헷갈리기 쉬운 점**
NULL은 `= NULL`로 비교할 수 없다. NULL은 "값이 없음"이라는 특수 상태라서 `=`로는 걸러지지 않고, 반드시 `IS NULL` / `IS NOT NULL`을 써야 한다. `WHERE x = NULL`은 아무것도 못 찾는다.

**5. 한 줄 정리**
빈 값은 `IS NULL`, 값이 있는 것은 `IS NOT NULL`로 거른다. `= NULL`은 동작하지 않는다.

## 코드로 보기 — LEFT JOIN + IS NULL로 "짝 없는 데이터" 찾기

```sql
-- 17번 유형: 아티스트 전체에 앨범을 LEFT JOIN
SELECT artists.name, albums.title
FROM artists
LEFT JOIN albums
ON artists.artist_id = albums.artist_id;

-- 18번 유형: 그중 앨범이 없는 아티스트만 (NULL 활용)
SELECT artists.name
FROM artists
LEFT JOIN albums
ON artists.artist_id = albums.artist_id
WHERE albums.album_id IS NULL;
```

**코드목적**
첫 쿼리는 "모든 아티스트와 (있으면) 그 앨범"을 보여 주고, 둘째 쿼리는 거기서 "앨범이 하나도 없는 아티스트"만 추려 내는 것이 목적이다.

**해석**
LEFT JOIN이라 앨범이 없는 아티스트도 첫 결과에 남는다(앨범 컬럼은 NULL). 둘째 쿼리에서 그 NULL 행만 `IS NULL`로 거르면, 정확히 "앨범 없는 아티스트"가 나온다. INNER JOIN이었다면 이들은 애초에 결과에서 사라져 찾을 수 없었을 것이다. 바로 이 점이 LEFT JOIN을 쓰는 이유다.

**실무 연결**
"구매 이력이 없는 가입 회원", "한 번도 로그인하지 않은 계정", "담당자가 배정되지 않은 문의"처럼 **"있어야 할 짝이 비어 있는 것"** 을 찾는 일은 데이터 품질 점검과 마케팅 타깃 추출에서 매우 자주 쓰인다. 그 핵심 도구가 LEFT JOIN + IS NULL이다.

> 참고 — 데이터 정의서(Data Dictionary): `customer` 테이블의 `support_rep_id` 같은 컬럼은 이름만 봐서는 의미가 모호하다. 이 값이 "고객 담당 직원(`employee_id`)"을 가리킨다는 사실을 추론 없이 알려 주는 문서가 데이터 정의서다. 정의서가 있으면 어떤 컬럼을 어떤 키로 조인해야 할지 헷갈리지 않는다. 실무에서 조인을 정확히 하려면 테이블 관계도(ERD)와 함께 꼭 챙겨야 할 자료다.

## 직접 해보기

1. `artists`를 기준으로 `albums`를 LEFT JOIN해, 모든 아티스트와 (있으면) 앨범 제목을 출력해 보자.
2. 위 결과에서 "앨범이 없는 아티스트"만 골라내 보자. (어떤 컬럼에 `IS NULL`을 걸어야 할까?)
3. `RIGHT JOIN`으로 쓴 쿼리를, 테이블 순서를 바꿔 같은 결과의 `LEFT JOIN`으로 바꿔 보자.

<details>
<summary>예시 답안 보기</summary>

```sql
-- 1번
SELECT artists.name, albums.title
FROM artists
LEFT JOIN albums
ON artists.artist_id = albums.artist_id;

-- 2번 (앨범 쪽 키가 비어 있는 행)
SELECT artists.name
FROM artists
LEFT JOIN albums
ON artists.artist_id = albums.artist_id
WHERE albums.album_id IS NULL;

-- 3번: A RIGHT JOIN B  →  B LEFT JOIN A
-- (예) FROM user RIGHT JOIN rental  ≡  FROM rental LEFT JOIN user
SELECT *
FROM rental
LEFT JOIN user
ON user.id = rental.user_id;
```

</details>

## 헷갈리기 쉬운 포인트

- **INNER JOIN vs LEFT JOIN**: "양쪽 다 있는 것만"은 INNER, "한쪽 전체 + 있으면 붙이기"는 LEFT.
- **LEFT vs RIGHT**: 기준이 왼쪽 테이블이면 LEFT, 오른쪽이면 RIGHT. 헷갈리면 순서를 바꿔 LEFT로 통일한다.
- **= NULL vs IS NULL**: NULL 비교는 반드시 `IS NULL` / `IS NOT NULL`. `= NULL`은 아무것도 못 찾는다.

## 연결되는 개념

- 이전 글: [INNER JOIN](03-inner-join.md) — 교집합 조인을 먼저 이해해야 "한쪽 전체 기준"의 의미가 와닿는다.
- 다음 글: [서브쿼리](05-subquery.md) — "짝 없는 데이터 찾기"는 서브쿼리(`NOT IN` 등)로도 풀 수 있어, 두 방식을 비교하게 된다.
- 더 찾아볼 키워드: OUTER JOIN, FULL OUTER JOIN, `COALESCE`(NULL을 기본값으로 치환), 데이터 정의서·ERD

## 셀프 체크

- [ ] LEFT JOIN이 "왼쪽 테이블 전체를 남긴다"는 것을 설명할 수 있다.
- [ ] 매칭되지 않은 행이 NULL로 표시되는 이유를 안다.
- [ ] RIGHT JOIN을 LEFT JOIN으로 바꾸는 방법(순서 교환)을 안다.
- [ ] `IS NULL` / `IS NOT NULL`의 쓰임을 구분할 수 있다.
- [ ] LEFT JOIN + IS NULL로 "짝 없는 데이터"를 찾는 패턴을 안다.

**복습 질문 및 답변**

*기본* — "대여 기록이 없는 회원도 포함"하려면 어떤 조인을 써야 하나?
> 회원 테이블을 왼쪽에 두고 LEFT JOIN한다.

*이해 확인* — `WHERE albums.album_id IS NULL`은 어떤 행을 남기나?
> LEFT JOIN 결과에서 매칭되는 앨범이 없어 NULL이 된 행, 즉 "앨범이 없는 아티스트"를 남긴다.

*응용* — `FROM user RIGHT JOIN rental`과 같은 결과를 LEFT JOIN으로 쓰면?
> 테이블 순서를 바꿔 `FROM rental LEFT JOIN user`로 쓰면 된다.

## 한 줄 정리

> LEFT JOIN은 한쪽(왼쪽) 테이블을 통째로 남기고 짝이 없으면 NULL로 채우며, 이 NULL을 `IS NULL`로 골라내면 "짝 없는 데이터"를 정확히 찾을 수 있다. RIGHT JOIN은 보통 순서를 바꿔 LEFT로 대체한다.
