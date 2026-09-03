# MongoDB와 Mongoose

문서형 데이터 모델부터 MongoDB 핵심 구조, Mongoose Schema·CRUD·Populate, Express 연결 수명주기와 ORM 비교까지 데이터 계층의 기반을 학습합니다.

## 학습 목표

- 관계형 모델과 문서형 모델의 차이를 설명합니다.
- MongoDB의 Database, Collection, Document와 ObjectId를 구분합니다.
- Mongoose Schema와 Model로 검증 가능한 데이터 구조를 만듭니다.
- 안전한 필터와 옵션으로 CRUD·Populate를 수행합니다.
- Express 연결 수명주기와 데이터 기술 선택 기준을 설계합니다.

## 추천 학습 순서

1. [문서형 데이터베이스 기초](01-document-database-foundations.md)
2. [MongoDB 핵심 구조와 운영 방식](02-mongodb-core-concepts-and-operations.md)
3. [Mongoose Schema와 Model](03-mongoose-schema-and-model.md)
4. [Mongoose CRUD와 Query](04-mongoose-crud-and-queries.md)
5. [문서 관계와 Populate](05-document-relations-and-populate.md)
6. [Express와 Mongoose 연결 관리](06-express-mongoose-integration.md)
7. [ODM과 ORM 선택](07-odm-and-orm-selection.md)
8. [GLOSSARY](GLOSSARY.md)

## 전체 흐름

```text
접근 패턴 분석 → 포함·참조 모델링 → Schema·Model 정의
→ 연결 준비 → 검증된 CRUD → Populate와 관계 정책
→ 인덱스·관측·마이그레이션·안전한 종료
```

## 주제별 빠른 찾기

| 궁금한 내용 | 학습 페이지 |
|---|---|
| RDB와 Document DB 비교 | 01 문서형 데이터베이스 기초 |
| Collection, ObjectId, Compass | 02 MongoDB 핵심 구조와 운영 방식 |
| Schema, Model, validation | 03 Mongoose Schema와 Model |
| create, find, update, delete | 04 Mongoose CRUD와 Query |
| ref, populate, 관계 무결성 | 05 문서 관계와 Populate |
| connect, 이벤트, 안전한 종료 | 06 Express와 Mongoose 연결 관리 |
| Mongoose와 Sequelize 비교 | 07 ODM과 ORM 선택 |

## 최종 점검

- [ ] 접근 패턴에 따라 포함과 참조를 선택한다.
- [ ] 연결 정보와 운영 계정을 비밀로 관리한다.
- [ ] 필터, 반환값과 업데이트 검증을 확인한다.
- [ ] Populate의 필드와 비용을 제한한다.
- [ ] Schema 변경에 migration과 복구 계획을 둔다.
