# 제약 조건 — 잘못된 데이터가 들어오지 못하게 막는 마지막 방어선

> 화면에서 막고, 서버에서 한 번 더 막아도, 결국 데이터가 닿는 마지막 관문은 데이터베이스입니다. 그 관문에 세워 두는 규칙이 바로 제약 조건입니다.

`제약조건` `NOT NULL` `UNIQUE` `DEFAULT` `CHECK` `CONSTRAINT` `무결성`

## 핵심요약

- 제약 조건은 **테이블에 잘못된 데이터가 입력되는 것을 막기 위한 규칙** 이며, 이를 통해 데이터의 무결성을 지킨다.
- 제약 조건은 프론트엔드·백엔드의 실수까지 대비하는 **"마지막 방어선"** 역할을 한다.
- `NOT NULL` 은 빈 값을, `UNIQUE` 는 중복 값을 막는다.
- `DEFAULT` 는 값이 없을 때 기본값을 자동으로 채우고, `CHECK` 는 값의 범위를 제한한다.
- `CONSTRAINT` 키워드로 제약 조건에 **이름** 을 붙이면 조회·관리가 쉬워진다.
- 제약 조건은 `ALTER TABLE` 로 나중에 추가하거나 삭제할 수 있다.
- "쓸모없는 데이터를 줄이는 것" 자체가 분석·개발 비용을 낮추는 일이다.

## 개념별 정리

### 제약 조건이란 (왜 DB 레벨에서 막는가)

**1. 정의**
제약 조건은 테이블에 들어오는 데이터가 지켜야 할 규칙입니다. 규칙에 어긋나는 데이터는 입력 단계에서 에러로 거부됩니다.

**2. 왜 필요한가?**
화면(프론트엔드)에서 입력값을 검사하고, 서버(백엔드)에서 한 번 더 검사해도 실수는 발생합니다. 비즈니스 규칙을 DB 레벨에서 강제하면, 앞단의 모든 검증이 뚫려도 잘못된 데이터가 끝내 저장되지 않습니다. 관점별로 정리하면:

- **DB 관리자 관점**: 무결성을 유지한다.
- **비즈니스 관점**: 규칙을 강제한다 (예: 19세 미만 가입 차단).
- **분석/개발 관점**: 불필요한 데이터가 줄어, 쿼리로 탐색·정제하는 비용이 낮아지고 양질의 데이터를 확보한다.

**3. 예시**
`CREATE TABLE` 안에서 각 속성 뒤에 제약 조건을 붙입니다. 기본 형태는 `CREATE TABLE 테이블명(속성1 데이터타입1 제약조건1, ...);` 입니다.

```sql
CREATE TABLE customer(
    id      VARCHAR(10) NOT NULL,
    name    VARCHAR(10) NOT NULL,
    address VARCHAR(30) NULL
);
```

**4. 헷갈리기 쉬운 점**
누가 "쓸모없는 데이터"인지 판단하는 주체는 분석가나 개발자가 아니라 **도메인 지식을 가진 사람** 입니다. 규칙끼리 충돌하면 테이블을 수정(`ALTER TABLE`)해 조정합니다.

**5. 한 줄 정리**
제약 조건은 데이터가 들어오는 입구에 세운 "출입 규칙"이다.

### NOT NULL

**1. 정의**
`NOT NULL` 은 널(빈) 값을 허용하지 않는 제약 조건입니다. 즉 "필수 입력" 조건입니다.

**2. 왜 필요한가?**
이름 없는 회원, 가격 없는 상품처럼 반드시 있어야 하는 값이 비어 버리면 데이터로서 의미가 없습니다.

**3. 예시**

```sql
CREATE TABLE customer(
  id   VARCHAR(10),
  name VARCHAR(10) NOT NULL
);

INSERT INTO customer (id, name) VALUES('kmax6', '김민준');
INSERT INTO customer (name) VALUES('이서연');

-- 에러 발생: name이 NOT NULL인데 값을 주지 않음
-- INSERT INTO customer (id) VALUES('kmax6');
```

**4. 헷갈리기 쉬운 점**
아무 제약 조건도 명시하지 않으면 기본값은 "널 값 허용"입니다. 그래서 `name` 처럼 꼭 필요한 값에는 명시적으로 `NOT NULL` 을 붙여야 합니다.

**5. 한 줄 정리**
`NOT NULL` 은 "이 칸은 비워 두면 안 된다"는 강제 규칙이다.

### UNIQUE

**1. 정의**
`UNIQUE` 는 중복되는 값을 허용하지 않는 제약 조건입니다.

**2. 왜 필요한가?**
이메일, 사업자번호처럼 "한 사람에 하나뿐이어야 하는" 값이 중복되면 데이터가 꼬입니다.

**3. 예시**

```sql
CREATE TABLE customer(
    id   VARCHAR(10) UNIQUE,
    name VARCHAR(10) NOT NULL
);

INSERT INTO customer (id, name) VALUES('kmax6', '김민준');
-- 에러 발생: id 'kmax6'가 이미 존재함
-- INSERT INTO customer (id, name) VALUES('kmax6', '이서연');

INSERT INTO customer (name) VALUES('김민준');
INSERT INTO customer (name) VALUES('이서연');
```

**4. 헷갈리기 쉬운 점**
`NULL` 값은 서로 비교가 불가능하므로, `UNIQUE` 컬럼이라도 `NULL` 은 여러 번 들어갈 수 있습니다(에러가 나지 않음). 또 이메일은 `UNIQUE` 가 적합하지만, 이름은 동명이인이 있어 일반적으로 부적합합니다.

**5. 한 줄 정리**
`UNIQUE` 는 "같은 값은 단 하나만"이라는 규칙이며, 빈 값(NULL)은 예외다.

### DEFAULT

**1. 정의**
`DEFAULT` 는 값을 지정하지 않았을 때 자동으로 채워질 기본값을 정합니다.

**2. 왜 필요한가?**
모든 칸을 사용자가 직접 채우게 하면 번거롭고 누락도 잦습니다. "입력 안 하면 이 값으로 둔다"를 정해 두면 편리합니다.

**3. 예시**

```sql
CREATE TABLE customer(
    id      VARCHAR(10) UNIQUE,
    name    VARCHAR(10) NOT NULL,
    address VARCHAR(30) DEFAULT 'No Address'
);

-- address가 'No Address'로 설정됨
INSERT INTO customer (id, name) VALUES('kmax6', '김민준');

-- address가 '서울시 동작구 대방동'으로 설정됨
INSERT INTO customer VALUES('flykite', '이서연', '서울시 동작구 대방동');
```

**4. 헷갈리기 쉬운 점**
`DEFAULT` 가 항상 정답은 아닙니다. 예컨대 쇼핑몰 배송지 주소는 기본값을 채우기보다 `NOT NULL` 로 반드시 입력받는 편이 더 안전할 수 있습니다. 제약 조건 선택은 비즈니스 로직에 따라 달라집니다.

**5. 한 줄 정리**
`DEFAULT` 는 "안 적으면 이걸로 채워 둘게"라는 친절한 기본값이다.

### CHECK

**1. 정의**
`CHECK` 는 값의 범위를 제한하여 특정 조건을 만족하는 값만 허용합니다.

**2. 왜 필요한가?**
나이는 음수가 될 수 없고, 성인 서비스는 19세 이상만 가입할 수 있습니다. 이런 "값의 범위 규칙"을 DB가 직접 검사하게 만듭니다.

**3. 예시**

```sql
CREATE TABLE customer(
    id      VARCHAR(10) UNIQUE,
    name    VARCHAR(10) NOT NULL,
    address VARCHAR(30) DEFAULT 'No Address',
    age     INT         CHECK (age >= 19)
);

INSERT INTO customer VALUES('kmax6', '김민준', '서울시 관악구 신림동', 20);

-- 에러 발생: age가 19 미만(18)
-- INSERT INTO customer VALUES('flykite', '이서연', '서울시 동작구 대방동', 18);
```

**4. 헷갈리기 쉬운 점**
`CHECK` 는 "조건을 만족하지 않으면 거부"이지 "조건에 맞게 고쳐 주는" 게 아닙니다. 18을 넣으면 19로 바꿔 주는 게 아니라 그냥 에러가 납니다.

**5. 한 줄 정리**
`CHECK` 는 "이 범위 안의 값만 통과"라는 검문소다.

### CONSTRAINT — 제약 조건에 이름 붙이기

**1. 정의**
`CONSTRAINT` 키워드로 제약 조건에 이름을 부여할 수 있습니다. 형태는 `CONSTRAINT 제약조건이름 제약조건[UNIQUE, CHECK, ...] (적용할 속성);` 입니다.

**2. 왜 필요한가?**
이름이 없으면 나중에 특정 제약 조건만 골라 수정·삭제하기 어렵습니다. 이름을 붙이면 조회와 관리가 쉬워집니다.

**3. 예시**

```sql
CREATE TABLE customer(
    id    VARCHAR(10),
    age   INT,
    CONSTRAINT id_unique UNIQUE (id),
    CONSTRAINT age_check CHECK (age >= 19)
);
```

생성된 제약 조건은 다음으로 확인합니다.

```sql
SELECT * FROM information_schema.table_constraints;
```

**4. 헷갈리기 쉬운 점**
이름을 붙이는 방식은 속성 뒤에 직접 쓰는 방식과 결과가 같습니다. 다만 이름이 있어야 나중에 `DROP CONSTRAINT 이름` 으로 정확히 지목해 삭제할 수 있습니다.

**5. 한 줄 정리**
`CONSTRAINT` 이름은 규칙에 붙인 명찰이라, 나중에 부르기 쉽다.

### 제약 조건 추가·삭제 (ALTER TABLE)

**1. 정의**
이미 만든 테이블에도 `ALTER TABLE` 로 제약 조건을 추가하거나 삭제할 수 있습니다.

**2. 왜 필요한가?**
운영 중에 정책이 바뀌면 제약 조건도 바뀌어야 합니다. 테이블을 새로 만들지 않고 규칙만 조정할 수 있어야 합니다.

**3. 예시**

```sql
-- 제약 조건 추가
ALTER TABLE customer
ADD CONSTRAINT address_unique UNIQUE (address);

ALTER TABLE customer
ADD CONSTRAINT customer_chk_2 CHECK (age >= 19 AND name = '김민준');

-- DEFAULT 제약 조건 수정
ALTER TABLE customer
ALTER address SET DEFAULT '주소 없음';
```

```sql
-- 제약 조건 삭제
ALTER TABLE customer DROP CONSTRAINT customer_chk_2;

-- DEFAULT 제약 조건 삭제
ALTER TABLE customer
ALTER address DROP DEFAULT;
```

**4. 헷갈리기 쉬운 점**
일반 제약 조건은 `ADD CONSTRAINT 이름` / `DROP CONSTRAINT 이름` 으로 다루지만, `DEFAULT` 만은 `ALTER 속성 SET DEFAULT 값` / `ALTER 속성 DROP DEFAULT` 처럼 별도 문법을 씁니다.

**5. 한 줄 정리**
규칙도 살아 있어서, 운영 중에 붙였다 뗐다 할 수 있다.

## 코드로 보기 — 제약 조건을 모두 얹은 테이블

```sql
CREATE TABLE customer(
    id      VARCHAR(10) UNIQUE,                  -- 중복 불가
    name    VARCHAR(10) NOT NULL,                -- 필수 입력
    address VARCHAR(30) DEFAULT 'No Address',    -- 미입력 시 기본값
    age     INT         CHECK (age >= 19)        -- 19세 이상만 허용
);
```

**코드목적**
네 가지 제약 조건이 한 테이블 안에서 어떻게 역할을 나눠 데이터 품질을 지키는지 보여 줍니다.

**해석**
이 테이블에 데이터를 넣으려 하면 DB가 네 개의 관문을 차례로 검사합니다. id가 겹치면(UNIQUE), name이 비면(NOT NULL), age가 18 이하면(CHECK) 입력이 거부되고, address를 안 적으면 자동으로 'No Address'가 채워집니다(DEFAULT). 결과적으로 "규칙을 지킨 데이터"만 테이블에 남습니다.

**실무 연결**
회원 가입, 주문, 결제 같은 핵심 테이블일수록 제약 조건이 촘촘합니다. 데이터 분석가 입장에서도 제약 조건이 잘 잡힌 테이블은 결측·중복·이상치가 적어, 전처리 시간이 크게 줄어듭니다.

## 직접 해보기

1. `product` 테이블을 만들되, `code` 는 중복 불가, `name` 은 필수 입력, `stock`(재고)은 0 이상만 허용하도록 제약 조건을 붙여 보세요.
2. 위 `product` 테이블에 `category` 컬럼이 있다고 가정하고, 값을 안 넣으면 '미분류'가 되도록 `DEFAULT` 를 `ALTER TABLE` 로 추가해 보세요.
3. 이메일을 저장하는 컬럼에는 `UNIQUE` 가 적합하지만 이름 컬럼에는 부적합한 이유를 한 문장으로 설명해 보세요.

## 헷갈리기 쉬운 포인트

- **NOT NULL vs UNIQUE**: 전자는 "비어 있으면 안 됨", 후자는 "중복되면 안 됨". `UNIQUE` 는 NULL 중복을 허용하지만 `NOT NULL` 은 NULL 자체를 막는다.
- **DEFAULT vs NOT NULL**: 미입력 시 자동으로 채울지(DEFAULT) vs 미입력 자체를 거부할지(NOT NULL). 상황에 따라 선택.
- **CHECK vs 애플리케이션 검증**: 화면 검증은 사용자 편의, `CHECK` 는 최후의 강제. 둘 다 있어야 안전하다.

## 연결되는 개념

- 이전에 알면 좋은 개념: [관계형 DB 구조와 SQL 기본](02-relational-db-and-sql.md) — `CREATE TABLE` 의 기본을 알아야 제약 조건을 얹을 수 있습니다.
- 다음에 이어지는 개념: [키와 무결성](04-keys-and-integrity.md) — 기본키·외래키도 사실은 특별한 제약 조건입니다.
- 더 찾아볼 키워드: `information_schema`, `트리거(trigger)`, `데이터 정합성`

## 셀프 체크

- [ ] 제약 조건이 "마지막 방어선"인 이유를 설명할 수 있다.
- [ ] `NOT NULL`·`UNIQUE`·`DEFAULT`·`CHECK` 의 차이를 구분할 수 있다.
- [ ] `CONSTRAINT` 로 제약 조건에 이름을 붙이는 이유를 안다.
- [ ] `ALTER TABLE` 로 제약 조건을 추가·삭제할 수 있다.

**복습 질문 및 답변**

- (기본) `NOT NULL` 과 `UNIQUE` 의 차이는?
  → `NOT NULL` 은 빈 값을 막고, `UNIQUE` 는 중복 값을 막는다.
- (이해확인) `UNIQUE` 컬럼에 `NULL` 이 여러 번 들어가도 에러가 나지 않는 이유는?
  → `NULL` 은 서로 비교가 불가능해 "중복"으로 판단되지 않기 때문이다.
- (응용) "성인 인증이 필요한 서비스에서 19세 미만 가입을 DB 차원에서 막고 싶다." 어떤 제약 조건을 어떻게 쓰면 될까?
  → `age INT CHECK (age >= 19)` 처럼 `CHECK` 제약 조건으로 값의 범위를 제한한다.

## 한 줄 정리

> 제약 조건은 데이터가 들어오는 입구에서 규칙을 강제하는 마지막 방어선이며, `NOT NULL`(필수)·`UNIQUE`(중복 금지)·`DEFAULT`(기본값)·`CHECK`(범위 제한)를 비즈니스 로직에 맞게 골라 쓴다.
