# ODM과 ORM 선택

Mongoose와 Sequelize 같은 도구는 데이터 접근을 추상화하지만 기반 데이터 모델의 차이까지 없애 주지는 않습니다.

**핵심 키워드:** ODM, ORM, Mongoose, Sequelize, 관계

## 핵심 내용

- ODM은 객체와 문서형 데이터 모델을 연결합니다.
- ORM은 객체와 관계형 테이블·행을 연결합니다.
- Mongoose는 MongoDB 문서 Schema, Model, Query와 Populate를 제공합니다.
- Sequelize 같은 ORM은 관계형 데이터베이스의 모델, 관계와 쿼리 API를 제공합니다.
- 도구보다 데이터 관계, 트랜잭션, 쿼리 패턴과 운영 역량을 먼저 판단합니다.

## ODM과 ORM 비교

| 관점 | ODM | ORM |
|---|---|---|
| 기반 모델 | 문서·컬렉션 | 테이블·행·관계 |
| 대표 관계 표현 | 포함, 참조 | 외래 키, JOIN |
| 구조 관리 | 문서 Schema와 검증 | 테이블 Schema와 migration |
| 예시 도구 | Mongoose | Sequelize |

## 관계형 모델 예시

```javascript
User.hasMany(Post, { foreignKey: "authorId" });
Post.belongsTo(User, { foreignKey: "authorId" });
```

- **목적:** 사용자 한 명과 여러 게시글의 관계를 ORM 모델에 선언합니다.
- **흐름:** Model 정의 → 외래 키 관계 선언 → migration → include 또는 별도 쿼리입니다.
- **결과:** 객체 API로 관계형 데이터를 조회할 수 있습니다.
- **실무 포인트:** 관계 선언이 실제 데이터베이스 migration을 자동으로 안전하게 수행한다는 뜻은 아닙니다.

## 쿼리와 포함

Mongoose의 Populate와 ORM의 include는 겉으로 비슷하게 관련 데이터를 가져오지만 내부 동작과 성능 특성이 다릅니다. 생성되는 쿼리와 인덱스, 반환 행 수를 실제로 확인해야 합니다.

## Schema 동기화 주의

개발 중 자동 동기화는 편리하지만 운영에서 파괴적인 Schema 변경을 만들 수 있습니다. 명시적 migration, 검토, 백업과 롤백 계획을 사용합니다.

## 선택 기준

- 데이터가 자연스럽게 문서 단위로 읽히고 변화가 잦은가?
- 복잡한 관계, 집계와 강한 무결성 제약이 핵심인가?
- 다중 데이터 변경의 트랜잭션 요구가 무엇인가?
- 팀이 운영·모니터링·튜닝할 수 있는 기술은 무엇인가?
- 장기적인 migration과 분석 요구는 무엇인가?

## 실습

1. 주문·결제 시스템과 콘텐츠 초안 저장 시스템에 적합한 모델을 비교하세요.
2. Populate와 JOIN 기반 include의 공통점과 차이를 적으세요.
3. 운영에서 자동 Schema sync를 주의해야 하는 이유를 설명하세요.

<details>
<summary>답</summary>

주문·결제는 강한 관계와 무결성, 트랜잭션 요구를 먼저 검토하고 콘텐츠 초안은 문서 구조의 유연성과 접근 패턴을 검토합니다. Populate와 include 모두 관련 데이터를 편리하게 표현하지만 기반 데이터 모델과 실행 쿼리가 다릅니다.

</details>

## 더 알아보기

- [문서형 데이터베이스 기초](01-document-database-foundations.md)
- [Express와 Mongoose 연결 관리](06-express-mongoose-integration.md)

## 체크리스트

- [ ] ODM과 ORM의 기반 모델을 구분한다.
- [ ] 데이터 관계와 트랜잭션을 먼저 분석한다.
- [ ] 추상화가 만든 실제 쿼리를 확인한다.
- [ ] 운영 Schema 변경에 migration을 사용한다.
- [ ] 팀의 운영 역량과 장기 요구를 반영한다.

## 복습 질문 및 답변

### Q1. ODM과 ORM은 같은 데이터베이스에 이름만 다르게 사용하는 도구인가요?

<details>
<summary>답</summary>

아닙니다. ODM은 주로 문서형 모델을, ORM은 관계형 모델을 객체 코드에 연결하며 기반 구조와 관계 처리 방식이 다릅니다.

</details>

### Q2. ORM을 사용하면 SQL과 인덱스를 몰라도 되나요?

<details>
<summary>답</summary>

아닙니다. 추상화 아래에서 실행되는 쿼리, JOIN, 트랜잭션과 인덱스를 이해해야 성능과 무결성 문제를 해결할 수 있습니다.

</details>

### Q3. 도구 기능이 많은 쪽을 선택하면 되나요?

<details>
<summary>답</summary>

기능 수보다 데이터 모델, 쿼리와 트랜잭션 요구, 운영 환경과 팀 경험에 맞는지를 기준으로 선택해야 합니다.

</details>

## 요약

ODM과 ORM은 서로 다른 데이터 모델의 생산성을 높이는 도구입니다. 추상화 이름보다 실제 저장 구조, 쿼리, 무결성과 운영 수명주기를 기준으로 선택하세요.
