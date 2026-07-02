# DCL과 인덱스 — 권한 관리와 조회 속도

> 데이터베이스를 만들었으면 이제 두 가지가 남습니다. "누가 무엇을 할 수 있게 할지"(권한)와 "어떻게 빨리 찾을지"(속도). 전자는 DCL로, 후자는 인덱스로 다룹니다.

`DCL` `GRANT` `REVOKE` `CREATE USER` `FLUSH PRIVILEGES` `TCL` `COMMIT` `ROLLBACK` `인덱스`

## 핵심요약

- **DCL(데이터 제어어)** 은 데이터베이스 접근 권한을 관리하는 명령으로, `GRANT`(부여)와 `REVOKE`(회수)가 핵심이다.
- 사용자는 `CREATE USER`로 만들고, 권한을 바꾼 뒤에는 `FLUSH PRIVILEGES`로 적용한다.
- `COMMIT`(저장)·`ROLLBACK`(취소)은 트랜잭션을 제어하므로 따로 **TCL**로 분류하기도 한다.
- **인덱스(Index)** 는 검색 속도를 높이는 자료구조로, 책의 색인과 같은 역할을 한다.
- 인덱스는 조회를 빠르게 하지만 저장 공간과 관리 비용이라는 **단점**도 있다.
- 모든 데이터를 조회하거나 변경이 잦은 컬럼에는 인덱스가 오히려 비효율적이다.

## 개념별 정리

### 데이터 제어어 (DCL)

**1. 정의**
데이터베이스에 접근하는 **권한을 관리**하는 등 데이터를 제어하는 명령어 그룹입니다. 대표적으로 권한을 주는 `GRANT`와 거두는 `REVOKE`가 있습니다.

**2. 왜 필요한가?**
한 데이터베이스를 여러 사람·여러 애플리케이션이 쓸 때, 누구는 조회만, 누구는 수정까지 가능하도록 나눠야 안전합니다. 전부에게 모든 권한을 주면 실수나 사고 한 번에 데이터가 망가질 수 있습니다.

**3. 예시**
사용자를 만들고 권한을 부여한 뒤 적용합니다.

```sql
-- 1) 사용자 생성 (localhost = 로컬 접속만 허용, 외부 허용은 '%')
CREATE USER webuser@localhost IDENTIFIED BY '1234';

-- 2) 특정 데이터베이스의 모든 테이블에 모든 권한 부여
GRANT ALL PRIVILEGES ON shared_kickboard.* TO webuser@localhost;

-- 3) 변경한 권한 설정을 즉시 적용
FLUSH PRIVILEGES;
```

```sql
-- 부여된 권한 확인
SHOW GRANTS FOR webuser@localhost;
```

```text
+-----------------------------------------------------------------------+
| Grants for webuser@localhost                                          |
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `webuser`@`localhost`                           |
| GRANT ALL PRIVILEGES ON `shared_kickboard`.* TO `webuser`@`localhost` |
+-----------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

```sql
-- 권한 회수
REVOKE ALL ON shared_kickboard.* FROM webuser@localhost;
```

`ALL`은 모든 권한을 뜻하며, 필요에 따라 `SELECT`(조회), `INSERT`(삽입), `DELETE`(삭제)처럼 권한을 골라서 줄 수도 있습니다.

**4. 헷갈리기 쉬운 점**
권한을 바꾼 뒤 `FLUSH PRIVILEGES`를 빼먹으면 변경이 곧바로 반영되지 않을 수 있습니다. 또 `localhost`와 `%`는 "어디서 접속하는 사용자인가"를 구분하는 부분으로, 같은 이름이라도 접속 위치가 다르면 별개로 취급됩니다.

**5. 한 줄 정리**
DCL은 "누구에게 무엇을 허용할지"를 `GRANT`/`REVOKE`로 정하는 명령입니다.

> 비유: 건물 출입카드를 발급(GRANT)하고 회수(REVOKE)하며, 어떤 층까지 들어갈 수 있는지 정하는 것.

### 트랜잭션 제어 (TCL)

**1. 정의**
여러 작업을 하나의 묶음(트랜잭션)으로 다루며, 그 결과를 확정하거나 되돌리는 명령입니다. `COMMIT`은 작업을 저장하고, `ROLLBACK`은 작업을 취소해 이전 상태로 돌립니다.

**2. 왜 필요한가?**
"계좌 A에서 빼고 B에 넣기"처럼 여러 작업이 모두 성공해야만 의미 있는 경우, 중간에 실패하면 전체를 되돌려야 데이터가 어긋나지 않습니다.

**3. 예시**

```sql
-- 작업 묶음 시작 후
UPDATE account SET balance = balance - 1000 WHERE id = 'A';
UPDATE account SET balance = balance + 1000 WHERE id = 'B';

COMMIT;     -- 두 작업을 함께 확정 저장
-- 문제가 생겼다면 ROLLBACK; 으로 둘 다 취소
```

**4. 헷갈리기 쉬운 점**
`COMMIT`/`ROLLBACK`은 권한이 아니라 "작업의 확정/취소"를 다루므로, DCL과 구분해 **TCL(Transaction Control Language)** 로 따로 분류하기도 합니다.

**5. 한 줄 정리**
TCL은 묶음 작업을 통째로 저장(COMMIT)하거나 통째로 취소(ROLLBACK)합니다.

### 인덱스 (Index)

**1. 정의**
데이터베이스 테이블의 **검색 속도를 높이기 위한 자료구조**입니다. 책 뒤의 색인처럼, 원하는 데이터의 위치를 빠르게 찾아가게 해 줍니다.

**2. 왜 필요한가?**
인덱스가 없으면 데이터를 찾을 때 테이블의 모든 행을 처음부터 하나씩 확인해야 합니다. 데이터가 많아질수록 이 방식은 급격히 느려집니다.

**3. 예시**

```sql
-- 인덱스 추가: customer 테이블의 id 컬럼에
CREATE INDEX customer_id ON customer (id);
```

```text
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

```sql
SHOW INDEX FROM customer;        -- 인덱스 확인
ALTER TABLE customer DROP INDEX customer_index;   -- 인덱스 삭제
```

**4. 헷갈리기 쉬운 점**
"인덱스를 많이 걸수록 빠르다"는 오해가 흔합니다. 인덱스는 조회를 빠르게 하지만, 데이터를 넣고·고치고·지울 때마다 인덱스도 함께 갱신해야 하므로 변경이 잦은 컬럼에선 오히려 느려질 수 있습니다. 또 `SELECT * FROM 테이블`처럼 전체를 조회한다면 인덱스가 쓸모없습니다.

**5. 한 줄 정리**
인덱스는 검색을 빠르게 하는 색인이지만, 공짜가 아니라 공간·관리 비용이 따릅니다.

> 비유: 두꺼운 책에서 단어를 찾을 때, 색인을 보면 빠르지만 그 색인을 만들고 책이 바뀔 때마다 고쳐야 하는 수고가 듭니다.

## 코드로 보기 — 인덱스를 언제 쓰면 좋은가

```text
인덱스를 쓰면 좋은 경우
 • 규모가 큰 테이블
 • 데이터의 삽입·수정·삭제가 많지 않은 컬럼
 • WHERE 조건절·ORDER BY(정렬)·JOIN에 자주 쓰는 컬럼
 • 데이터의 중복도가 낮은 컬럼

인덱스의 단점
 • 인덱스를 관리하기 위한 추가 작업이 필요
 • 인덱스를 저장할 추가 공간이 필요
 • 경우에 따라 검색 성능이 오히려 저하될 수 있음
```

**코드목적**
인덱스를 "걸까 말까" 판단하는 기준을 정리합니다.

**해석**
핵심은 "자주 찾고, 자주 안 바뀌고, 값이 다양한" 컬럼에 거는 것입니다. 반대로 전체 조회만 하거나, 값이 거의 같거나(중복도 높음), 수정이 잦은 컬럼은 인덱스 효과가 적거나 손해입니다.

**실무 연결**
서비스가 느려질 때 "어떤 쿼리가 느린지" 찾아 그 WHERE/JOIN 컬럼에 인덱스를 거는 것이 성능 튜닝의 기본 중 하나입니다. 단, 무작정 거는 게 아니라 사용 패턴을 보고 선택합니다.

## 직접 해보기

1. `report` 사용자를 만들고 `shared_kickboard`에 대해 조회(SELECT) 권한만 주는 명령을 적어보세요.
2. 위에서 준 권한을 회수하는 명령은 무엇일까요?
3. 회원의 로그인 ID처럼 "검색에 자주 쓰고 값이 다 다른" 컬럼에 인덱스를 거는 것이 적절한 이유를 설명해보세요.

<details>
<summary>답안 보기</summary>

1. `CREATE USER report@localhost IDENTIFIED BY '비밀번호';` 후 `GRANT SELECT ON shared_kickboard.* TO report@localhost;` 그리고 `FLUSH PRIVILEGES;`
2. `REVOKE SELECT ON shared_kickboard.* FROM report@localhost;`
3. 로그인 ID는 조회에 자주 쓰이고(WHERE 조건), 값이 거의 겹치지 않아(중복도 낮음) 인덱스로 위치를 빠르게 좁힐 수 있기 때문.
</details>

## 헷갈리기 쉬운 포인트

- **GRANT vs REVOKE**: 권한을 주는 것 vs 거두는 것.
- **DCL vs TCL**: 권한 제어(GRANT/REVOKE) vs 트랜잭션 제어(COMMIT/ROLLBACK).
- **인덱스 있음 vs 없음**: 빠른 조회를 얻는 대신 공간·갱신 비용을 치름. 전체 조회에는 무의미.

## 연결되는 개념

- 이전 글: [MySQL 설치와 데이터베이스 만들기](04-mysql-setup-and-database.md)
- 처음으로: [이상 현상과 함수 종속](01-anomalies-and-functional-dependency.md)
- 더 찾아볼 키워드: `SQL 명령어 분류(DDL·DML·DCL·TCL)`, `B-Tree 인덱스`, `복합 인덱스`

## 셀프 체크

- [ ] `GRANT`와 `REVOKE`의 역할을 구분할 수 있다.
- [ ] 사용자 생성 후 권한 적용까지의 흐름을 안다.
- [ ] COMMIT/ROLLBACK이 무엇을 하는지 설명할 수 있다.
- [ ] 인덱스를 거는 것이 적절한 경우와 그렇지 않은 경우를 구분할 수 있다.

**복습 질문 및 답변**

- (기본) 권한을 부여하는 DCL 명령은? → `GRANT`.
- (이해확인) 인덱스의 단점 한 가지는? → 저장 공간이 추가로 필요하고, 데이터 변경 시 인덱스도 갱신해야 해 비용이 든다.
- (응용) 값이 'Y'/'N' 두 가지뿐인 컬럼에 인덱스를 거는 게 효과적일까? → 중복도가 매우 높아 인덱스로 범위를 거의 못 좁히므로 비효율적이다.

## 한 줄 정리

> DCL은 GRANT/REVOKE로 권한을 통제하고, 인덱스는 색인처럼 조회를 빠르게 하지만 공간·관리 비용을 고려해 선택적으로 써야 합니다.
