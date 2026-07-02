# 관계형 DB 구조와 SQL 기본 — 표 만들고, 넣고, 꺼내기

> 데이터베이스를 골랐다면 이제 진짜 시작입니다. 빈 표를 만들고(`CREATE`), 데이터를 채워 넣고(`INSERT`), 원하는 걸 꺼내 보는(`SELECT`) 세 동작이면 관계형 DB의 8할은 끝납니다.

`테이블` `속성` `튜플` `도메인` `SQL` `CREATE TABLE` `INSERT` `SELECT` `DDL` `DML` `DCL`

## 핵심요약

- 관계형 DB는 데이터를 **행(row)과 열(column)을 가진 테이블** 로 표현한다.
- 테이블의 한 칸 한 칸이 모여 **속성(열)·튜플(행)·도메인(값의 범위)** 이라는 논리 단위를 이룬다.
- 테이블을 만드는 명령은 `CREATE TABLE`, 데이터를 넣는 명령은 `INSERT INTO`, 꺼내는 명령은 `SELECT`다.
- SQL 문법은 대문자, 테이블·속성 이름은 소문자로 쓰는 것이 권장되며, 명령어 끝에는 세미콜론(`;`)을 붙인다.
- 각 속성은 `VARCHAR`, `INT`, `FLOAT`, `DATETIME` 같은 **데이터 타입** 을 가지며, 타입은 DBMS마다 조금씩 다르다.
- SQL은 역할에 따라 **DDL(구조 정의)·DML(데이터 조작)·DCL(권한 제어)** 세 갈래로 나뉜다.
- 만든 구조는 `ALTER TABLE` 로 수정하고 `DROP TABLE` 로 삭제한다.

## 개념별 정리

### 테이블의 구성 요소 (속성·튜플·도메인)

**1. 정의**
테이블은 행과 열로 구성됩니다. 열 하나하나는 **속성(attribute)** 으로, 데이터의 특성을 나타내는 가장 작은 논리적 단위입니다. 행 하나하나는 **튜플(tuple)** 로, 속성들이 모여 구성된 각각의 행을 의미합니다. 한 속성이 가질 수 있는 값의 집합을 **도메인(domain)** 이라고 합니다.

**2. 왜 필요한가?**
이 용어들은 관계형 DB를 설명하는 공용 언어입니다. "행/열"이라는 일상어와 "튜플/속성"이라는 이론 용어를 함께 알아 두면, 강의 자료나 공식 문서를 읽을 때 막히지 않습니다.

**3. 예시**
고객 테이블을 떠올려 봅시다.

| ID | 이름 | 주소 |
| --- | --- | --- |
| kmax6 | 김민준 | 서울시 관악구 신림동 |
| flykite | 이서연 | 서울시 동작구 대방동 |
| freeman123 | 박서준 | 서울시 관악구 신림동 |

- `ID`, `이름`, `주소` = 열(column) = 속성(attribute)
- `kmax6 / 김민준 / 서울시 관악구 신림동` 한 줄 = 행(row) = 튜플(tuple)
- `주소` 칸에 들어갈 수 있는 모든 값의 범위 = 도메인(domain)

**4. 헷갈리기 쉬운 점**
속성(열)은 "어떤 종류의 정보인가"를, 튜플(행)은 "하나의 대상에 대한 정보 묶음"을 가리킵니다. 세로로 읽으면 속성, 가로로 읽으면 튜플이라고 기억하면 쉽습니다.

**5. 한 줄 정리**
테이블은 표, 열은 항목, 행은 한 사람의 기록 카드다.

### 테이블 만들기 (CREATE TABLE)

**1. 정의**
`CREATE TABLE` 은 새 테이블의 구조를 정의하는 명령입니다. 기본 형태는 `CREATE TABLE 테이블명(속성1 데이터타입1, 속성2 데이터타입2, ...);` 입니다.

**2. 왜 필요한가?**
데이터를 넣기 전에 "어떤 칸을, 어떤 형식으로 둘 것인가"를 먼저 정해야 합니다. 집을 짓기 전에 방 개수와 용도를 정하는 설계 단계입니다.

**3. 예시**

```sql
CREATE TABLE customer(
    id      VARCHAR(10),
    name    VARCHAR(10),
    address VARCHAR(30)
);
```

만든 뒤에는 다음 명령으로 확인합니다.

```sql
SHOW TABLES;     -- 데이터베이스의 테이블 목록 확인
DESC customer;   -- customer 테이블의 구조 확인
```

**4. 헷갈리기 쉬운 점**
`CREATE TABLE` 직후의 테이블은 **데이터가 비어 있는 것이 정상** 입니다. 구조를 만든 것이지 데이터를 넣은 게 아니기 때문입니다.

**5. 한 줄 정리**
`CREATE TABLE` 은 데이터를 담을 빈 표의 틀을 짜는 일이다.

### 데이터 넣기·꺼내기 (INSERT / SELECT)

**1. 정의**
`INSERT INTO` 는 테이블에 한 행씩 데이터를 추가하고, `SELECT` 는 테이블에서 원하는 속성을 골라 출력합니다.

**2. 왜 필요한가?**
구조만 있고 데이터가 없으면 의미가 없습니다. 넣고(`INSERT`) 꺼내는(`SELECT`) 동작이 데이터베이스를 "살아 있는" 도구로 만듭니다.

**3. 예시**

```sql
INSERT INTO customer (id, name, address)
VALUES('kmax6', '김민준', '서울시 관악구 신림동');

-- 속성의 순서는 중요하지 않음 (이름을 적어 짝을 맞추면 됨)
INSERT INTO customer (name, address, id)
VALUES('이서연', '서울시 동작구 대방동', 'flykite');

-- 모든 속성을 순서대로 입력하는 경우 속성 목록은 생략 가능
INSERT INTO customer
VALUES('freeman123', '박서준', '서울시 관악구 신림동');
```

```sql
SELECT id, name, address FROM customer;

-- 출력하고 싶은 속성, 순서 조정 가능
SELECT address, name FROM customer;

-- "*"을 이용해 모든 속성 출력 가능
SELECT * FROM customer;
```

**4. 헷갈리기 쉬운 점**
`INSERT INTO customer (속성목록)` 에서 속성 이름을 함께 적으면 순서를 자유롭게 바꿔도 됩니다. 속성 목록을 생략하면 **테이블에 정의된 순서 그대로** 값을 넣어야 합니다. 또한 값을 넣지 않은 속성은 기본값인 `NULL`이 들어갑니다.

**5. 한 줄 정리**
`INSERT` 는 표에 줄을 추가하는 일, `SELECT` 는 표에서 보고 싶은 칸만 골라 보는 일이다.

### 데이터 타입

**1. 정의**
각 속성은 어떤 종류의 값을 담을지 데이터 타입으로 지정합니다.

**2. 왜 필요한가?**
타입을 정해 두면 "숫자 칸에 글자가 들어오는" 사고를 막고, 저장 공간도 효율적으로 씁니다. 잘못된 타입은 정보 유실로 이어집니다 — 예컨대 날짜를 그냥 문자열로 저장하면 "날짜 정렬"이나 "기간 계산"이 어려워집니다.

**3. 예시**

| 자료형 | 의미 |
| --- | --- |
| `VARCHAR(n)` | nBytes 크기의 가변 길이 문자열 데이터 |
| `INT` | 정수형 숫자 데이터(4Bytes) |
| `FLOAT` | 4Bytes 크기의 부동 소수점 데이터 |
| `DATETIME` | 날짜와 시간 형태의 데이터(YYYY-MM-DD HH:MM:SS) |

제공되는 데이터 타입은 DBMS마다 차이가 있을 수 있으므로, 의심스러우면 공식 문서를 확인하는 습관이 중요합니다.

**4. 헷갈리기 쉬운 점**
`VARCHAR(n)` 의 `n` 은 "넉넉하게 잡아야 하는" 값입니다. 사용자 입력이나 다국적 이름처럼 길이가 들쭉날쭉한 데이터의 확장성을 고려해 설계해야 합니다. 너무 짧게 잡으면 나중에 데이터가 잘립니다.

**5. 한 줄 정리**
데이터 타입은 "이 칸에는 이런 모양의 값만 들어온다"는 약속이다.

### SQL의 세 갈래 (DDL · DML · DCL)

**1. 정의**
SQL(Structured Query Language)은 관계형 DB를 다루는 표준 언어로, 역할에 따라 세 가지로 나뉩니다.

1. **DDL(Data Definition Language)**: 테이블과 같은 데이터 *구조* 를 정의 (예: `CREATE`, `ALTER`, `DROP`)
2. **DML(Data Manipulation Language)**: 데이터 *조회 및 조작* (예: `INSERT`, `SELECT`)
3. **DCL(Data Control Language)**: 데이터베이스에 접근하는 *권한 관리*

**2. 왜 필요한가?**
같은 SQL이라도 "틀을 바꾸는 명령"과 "데이터를 다루는 명령"은 성격이 완전히 다릅니다. 이 분류를 알면 명령어를 외우지 않아도 "이건 구조를 바꾸니 DDL이겠구나" 하고 위치를 잡을 수 있습니다.

**3. 예시**
이 글에서 다룬 `CREATE TABLE`·`ALTER TABLE`·`DROP TABLE` 은 모두 DDL, `INSERT`·`SELECT` 는 DML입니다.

**4. 헷갈리기 쉬운 점**
`INSERT`/`SELECT`(데이터 조작)와 `CREATE`/`DROP`(구조 정의)를 헷갈리지 마세요. 전자는 표 안의 *내용* 을, 후자는 표 자체의 *틀* 을 다룹니다.

**5. 한 줄 정리**
DDL은 그릇을 만드는 말, DML은 음식을 담고 꺼내는 말, DCL은 누가 손댈 수 있는지 정하는 말이다.

### 테이블 수정·삭제 (ALTER / DROP)

**1. 정의**
`ALTER TABLE` 은 이미 만든 테이블의 구조를 바꾸고, `DROP TABLE` 은 테이블 자체를 삭제합니다.

**2. 왜 필요한가?**
처음 설계가 완벽할 수는 없습니다. 컬럼을 추가하거나 이름을 바꾸거나 더 이상 안 쓰는 테이블을 정리하는 일은 실무에서 늘 일어납니다.

**3. 예시**

```sql
-- 컬럼 추가
ALTER TABLE customer ADD COLUMN birthday DATE NULL;
-- 컬럼 수정
ALTER TABLE customer MODIFY COLUMN id VARCHAR(15) NULL;
-- 컬럼 이름 변경
ALTER TABLE customer CHANGE COLUMN name korean_name VARCHAR(10) NOT NULL;
-- 컬럼 삭제
ALTER TABLE customer DROP COLUMN address;
-- 테이블 이름 변경
ALTER TABLE customer RENAME member;
```

```sql
DROP TABLE member;
```

**4. 헷갈리기 쉬운 점**
`MODIFY` 는 데이터 타입·제약 조건을 바꾸고, `CHANGE` 는 컬럼 *이름까지* 바꿉니다. 그래서 `CHANGE` 는 "기존 컬럼명 + 새 컬럼명"을 둘 다 적어야 합니다.

**5. 한 줄 정리**
`ALTER` 는 표를 리모델링, `DROP` 은 표를 통째로 철거하는 일이다.

## 코드로 보기 — 빈 표를 만들고 데이터를 채우는 한 흐름

```sql
-- 1단계: 표의 틀을 만든다 (DDL)
CREATE TABLE customer(
    id      VARCHAR(10),
    name    VARCHAR(10),
    address VARCHAR(30)
);

-- 2단계: 데이터를 채운다 (DML)
INSERT INTO customer VALUES('kmax6', '김민준', '서울시 관악구 신림동');

-- 3단계: 원하는 데이터를 꺼낸다 (DML)
SELECT * FROM customer;
```

**코드목적**
"구조 정의 → 데이터 입력 → 데이터 조회"라는 관계형 DB의 가장 기본적인 작업 흐름을 한눈에 보여 줍니다.

**해석**
1단계 후에는 빈 표가, 2단계 후에는 한 줄짜리 표가 생기고, 3단계 `SELECT *` 는 `kmax6 / 김민준 / 서울시 관악구 신림동` 한 행을 그대로 출력합니다. DDL로 그릇을 만들고 DML로 음식을 담고 꺼내는, 세 동작의 합주입니다.

**실무 연결**
신규 서비스에서 회원 테이블을 설계하고, 가입 데이터를 적재하고, 운영 화면에서 조회하는 전 과정이 정확히 이 흐름을 따릅니다. 데이터 분석가는 주로 3단계(`SELECT`)를, 백엔드 개발자는 1~2단계를 자주 다룹니다.

## 직접 해보기

1. `book` 이라는 테이블을 만들어 보세요. 속성은 `id`(문자 10자), `title`(문자 30자), `price`(정수)입니다. `CREATE TABLE` 문을 직접 써 보세요.
2. 위 `book` 테이블에 책 한 권을 `INSERT` 하는 문장을, (1) 속성 목록을 적는 방식과 (2) 생략하는 방식 두 가지로 써 보세요.
3. `book` 테이블에서 `title` 과 `price` 만 출력하는 `SELECT` 문을 써 보세요. 그다음 `price` 컬럼을 삭제하는 `ALTER TABLE` 문도 써 보세요.

## 헷갈리기 쉬운 포인트

- **속성(열) vs 튜플(행)**: 세로로 읽으면 속성, 가로로 읽으면 튜플.
- **DDL vs DML**: 표의 *틀* 을 다루면 DDL(`CREATE`/`ALTER`/`DROP`), 표의 *내용* 을 다루면 DML(`INSERT`/`SELECT`).
- **MODIFY vs CHANGE**: `MODIFY` 는 타입·제약 변경, `CHANGE` 는 이름까지 변경.
- **속성 목록 작성 vs 생략**: 적으면 순서 자유, 생략하면 정의된 순서 그대로.

## 연결되는 개념

- 이전에 알면 좋은 개념: [데이터베이스 개요](01-database-basics.md) — 왜 RDB를 쓰는지 알고 보면 테이블 구조가 더 잘 들어옵니다.
- 다음에 이어지는 개념: [제약 조건](03-constraints.md) — `CREATE TABLE` 에 규칙을 덧붙여 잘못된 데이터를 막는 법.
- 더 찾아볼 키워드: `WHERE`, `ORDER BY`, `JOIN`, `information_schema`

## 셀프 체크

- [ ] 속성·튜플·도메인을 테이블 예시로 설명할 수 있다.
- [ ] `CREATE TABLE`·`INSERT`·`SELECT` 의 기본 형태를 쓸 수 있다.
- [ ] `VARCHAR`·`INT`·`FLOAT`·`DATETIME` 의 쓰임을 구분할 수 있다.
- [ ] SQL을 DDL·DML·DCL로 분류할 수 있다.
- [ ] `ALTER TABLE` 로 컬럼을 추가·수정·삭제할 수 있다.

**복습 질문 및 답변**

- (기본) `SELECT * FROM customer;` 는 무엇을 하는 명령인가?
  → customer 테이블의 모든 속성(모든 열)을 모든 행에 대해 출력한다.
- (이해확인) `INSERT` 시 속성 목록을 생략하면 어떤 규칙을 지켜야 하는가?
  → 테이블에 정의된 속성 순서 그대로 값을 넣어야 한다.
- (응용) 운영 중인 `customer` 테이블에 "가입일" 정보를 추가로 저장해야 한다. 어떤 명령을 어떤 데이터 타입으로 써야 할까?
  → `ALTER TABLE customer ADD COLUMN signup_date DATETIME NULL;` 처럼 날짜·시간 타입(DATETIME)으로 컬럼을 추가한다.

## 한 줄 정리

> 관계형 DB는 행과 열의 테이블로 데이터를 표현하며, `CREATE`(틀 만들기)·`INSERT`(채우기)·`SELECT`(꺼내기) 세 동작과 DDL·DML·DCL이라는 분류만 잡으면 기본기가 선다.
