# MySQL 설치와 데이터베이스 만들기

> 종이에 그린 설계도(ERD)는 그 자체로는 아무 데이터도 담지 못합니다. 이제 실제 데이터베이스 관리 시스템인 MySQL을 설치하고, 설계한 구조를 올릴 그릇(데이터베이스)을 직접 만들어봅니다.

`MySQL` `패키지매니저` `CREATE DATABASE` `USE` `SHOW DATABASES` `DROP DATABASE` `서버`

## 핵심요약

- **MySQL**은 가장 널리 쓰이는 관계형 데이터베이스 관리 시스템(RDBMS) 중 하나다.
- 패키지 매니저로 설치하면 버전 관리와 설치가 간편하다. (윈도우=Chocolatey, 맥=Homebrew)
- 데이터베이스는 **서버를 켜고(start) → 접속하고 → 작업하고 → 종료(stop)** 하는 흐름으로 다룬다.
- `CREATE DATABASE`로 데이터베이스를 만들고, `USE`로 작업 대상을 지정한다.
- `SHOW DATABASES`로 목록을, `DROP DATABASE`로 삭제를 수행한다.
- "데이터베이스"는 여러 테이블을 담는 가장 큰 단위의 그릇이다.

## 개념별 정리

### MySQL 설치와 서버 다루기

**1. 정의**
MySQL은 데이터를 테이블 형태로 저장·관리하는 서버 프로그램입니다. 설치하면 "MySQL 서버"가 컴퓨터에서 돌아가고, 우리는 클라이언트로 거기에 접속해 명령을 보냅니다.

**2. 왜 필요한가?**
지금까지의 정규화·ERD는 설계 단계입니다. 실제로 데이터를 넣고 조회하려면 그것을 실행해 줄 DBMS가 필요하고, MySQL이 그 역할을 합니다.

**3. 예시**
패키지 매니저로 설치한 뒤, 서버를 켜고 접속합니다.

```bash
# 윈도우(Chocolatey)
choco install mysql

# 맥(Homebrew)
brew install mysql
```

```bash
# 서버 실행 / 접속 / 종료
net start mysql        # (윈도우) 서버 시작
mysql.server start     # (맥) 서버 시작

mysql -u root -p       # root 계정으로 접속 (초기 비밀번호 없으면 엔터)

# mysql> 프롬프트에서 작업한 뒤
\q                     # 접속 종료
net stop mysql         # (윈도우) 서버 종료
```

접속에 성공하면 프롬프트가 `mysql>` 로 바뀌고, 여기서부터 SQL 명령을 입력할 수 있습니다.

**4. 헷갈리기 쉬운 점**
"서버 실행"과 "접속"은 다른 단계입니다. 서버는 컴퓨터 뒤에서 도는 프로그램을 켜는 것이고(`start`), 접속은 그 서버에 클라이언트로 들어가는 것(`mysql -u root -p`)입니다. 서버가 꺼져 있으면 접속이 안 됩니다.

**5. 한 줄 정리**
MySQL은 서버를 켜고 접속해서 SQL로 데이터를 다루는 도구입니다.

> 비유: 서버 실행은 가게 문을 여는 것, 접속은 손님이 들어가는 것입니다.

### 데이터베이스 생성·사용·삭제

**1. 정의**
데이터베이스는 관련된 테이블들을 묶어 담는 가장 큰 단위입니다. 하나의 MySQL 서버 안에 여러 데이터베이스를 둘 수 있습니다.

**2. 왜 필요한가?**
테이블을 만들기 전에 그것을 담을 데이터베이스가 먼저 있어야 합니다. 프로젝트별로 데이터베이스를 분리하면 데이터가 섞이지 않습니다.

**3. 예시**

```sql
CREATE DATABASE shared_kickboard;   -- 데이터베이스 생성
```

```text
Query OK, 1 row affected (0.01 sec)
```

```sql
SHOW DATABASES;                     -- 목록 확인
```

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shared_kickboard   |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

방금 만든 `shared_kickboard`가 목록에 보입니다. 나머지는 MySQL이 기본 제공하는 시스템 데이터베이스입니다.

```sql
USE shared_kickboard;               -- 작업 대상으로 지정
```

```text
Database changed
```

`USE` 이후부터는 이 데이터베이스 안에서 테이블을 만들거나 데이터를 다룰 수 있습니다.

```sql
DROP DATABASE shared_kickboard;     -- 데이터베이스 삭제
```

```text
Query OK, 0 rows affected (0.03 sec)
```

**4. 헷갈리기 쉬운 점**
`USE`를 빼먹는 실수가 흔합니다. 데이터베이스를 만들었어도 `USE`로 지정하지 않으면 "어느 데이터베이스에 테이블을 만들지" 모르는 상태라 에러가 납니다. 또 `DROP DATABASE`는 안의 모든 테이블·데이터를 한 번에 지우므로 신중해야 합니다.

**5. 한 줄 정리**
`CREATE`로 만들고 `USE`로 들어가서 작업하며, 필요 없으면 `DROP`으로 지웁니다.

> 비유: 데이터베이스는 큰 서랍장, 테이블은 그 안의 서랍입니다. `USE`는 "이 서랍장을 쓰겠다"고 정하는 것.

## 코드로 보기 — 데이터베이스 한살이

```sql
-- 1) 만들기
CREATE DATABASE shared_kickboard;

-- 2) 잘 만들어졌는지 확인
SHOW DATABASES;

-- 3) 이 데이터베이스로 들어가서 작업 시작
USE shared_kickboard;
--    여기서 CREATE TABLE customer (...); 같은 작업을 이어감

-- 4) 더 이상 필요 없으면 통째로 삭제
DROP DATABASE shared_kickboard;
```

**코드목적**
데이터베이스를 만들고, 확인하고, 사용하고, 지우는 전체 생애 주기를 한 흐름으로 익힙니다.

**해석**
이 네 단계가 모든 DB 작업의 기본 골격입니다. 실제 프로젝트에서는 3번(`USE`) 이후에 정규화로 설계한 테이블들을 `CREATE TABLE`로 만들고 데이터를 채우게 됩니다.

**실무 연결**
백엔드 개발 초기에 가장 먼저 하는 일이 바로 이 데이터베이스 생성입니다. 서비스마다 별도 데이터베이스를 두고, 개발용·운영용을 분리하는 식으로 활용합니다.

## 직접 해보기

1. `my_blog`라는 데이터베이스를 만들고 목록에서 확인하는 명령을 적어보세요.
2. `my_blog`로 작업 대상을 바꾸는 명령은 무엇일까요?
3. 서버가 켜져 있지 않을 때 `mysql -u root -p`를 실행하면 왜 접속이 안 될지 설명해보세요.

<details>
<summary>답안 보기</summary>

1. `CREATE DATABASE my_blog;` 후 `SHOW DATABASES;`
2. `USE my_blog;`
3. 접속은 "돌고 있는 서버"에 들어가는 것인데, 서버가 꺼져 있으면 들어갈 대상이 없기 때문. 먼저 `net start mysql`(또는 `mysql.server start`)로 서버를 켜야 한다.
</details>

## 헷갈리기 쉬운 포인트

- **서버 실행 vs 접속**: `start`는 서버 켜기, `mysql -u root -p`는 그 서버에 들어가기.
- **데이터베이스 vs 테이블**: 데이터베이스는 테이블을 담는 큰 그릇, 테이블은 실제 데이터를 행·열로 담는 단위.
- **`DROP DATABASE` vs `DROP TABLE`**: 전자는 데이터베이스 전체, 후자는 테이블 하나만 삭제.

## 연결되는 개념

- 이전 글: [정규화 한눈에 정리](03-normalization-summary-denormalization.md)에서 설계한 스키마를 여기서 구현합니다.
- 다음 글: [DCL과 인덱스 — 권한 관리와 조회 속도](05-dcl-and-index.md)
- 더 찾아볼 키워드: `RDBMS`, `DDL(CREATE/DROP)`, `클라이언트-서버 구조`

## 셀프 체크

- [ ] MySQL 서버를 켜고 끄는 명령을 안다.
- [ ] 데이터베이스를 만들고 목록을 확인할 수 있다.
- [ ] `USE`의 역할을 설명할 수 있다.
- [ ] `DROP DATABASE`의 위험성을 안다.

**복습 질문 및 답변**

- (기본) 데이터베이스를 만드는 명령은? → `CREATE DATABASE 데이터베이스명;`
- (이해확인) `USE`를 하지 않으면 어떤 문제가 생기나? → 작업 대상 데이터베이스가 정해지지 않아 테이블 생성·조작 시 에러가 난다.
- (응용) 개발 중 데이터를 전부 초기화하고 처음부터 다시 만들고 싶다면 어떤 두 명령을 순서대로 쓸까? → `DROP DATABASE`로 지운 뒤 `CREATE DATABASE`로 다시 생성.

## 한 줄 정리

> MySQL은 서버를 켜고 접속한 뒤, `CREATE`·`USE`·`DROP`으로 데이터베이스를 만들고 사용하고 정리하는 흐름으로 다룹니다.
