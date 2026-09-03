# GLOSSARY

## 용어 정리

| 용어 | 설명 |
|---|---|
| NoSQL | 관계형 테이블 외의 다양한 데이터 모델을 사용하는 데이터베이스 계열 |
| Document Database | 필드와 값으로 구성된 문서를 기본 저장 단위로 사용하는 데이터베이스 |
| BSON | MongoDB가 문서를 저장할 때 사용하는 이진 직렬화 형식 |
| Database | 관련 컬렉션을 묶는 논리적 공간 |
| Collection | 비슷한 목적의 문서를 모아 저장하는 단위 |
| Document | MongoDB의 기본 데이터 저장 단위 |
| ObjectId | MongoDB 문서의 _id에 흔히 사용하는 고유 식별자 타입 |
| Embedding | 관련 데이터를 한 문서 안에 포함하는 모델링 방식 |
| Reference | 다른 문서의 식별자를 저장해 관계를 나타내는 방식 |
| ODM | 객체와 문서형 데이터 모델을 매핑하는 도구 |
| ORM | 객체와 관계형 테이블·행을 매핑하는 도구 |
| Mongoose | Node.js에서 MongoDB 모델과 쿼리를 제공하는 ODM |
| Schema | 필드 타입, 기본값, 검증과 인덱스 규칙의 정의 |
| Model | Schema를 기반으로 컬렉션 CRUD를 수행하는 인터페이스 |
| Document Instance | Model로 생성하거나 조회한 개별 문서 객체 |
| Validation | 데이터가 형식과 규칙을 만족하는지 확인하는 과정 |
| Index | 조회와 정렬을 빠르게 하기 위해 유지하는 별도 자료 구조 |
| Query Filter | 조회·수정·삭제할 문서를 선택하는 조건 |
| $in | 필드가 주어진 후보 값 중 하나인지 검사하는 연산자 |
| Populate | 참조 ID를 관련 Mongoose 문서로 조회해 결합하는 기능 |
| Migration | 데이터 또는 Schema 구조를 통제된 단계로 변경하는 절차 |
| Connection Pool | 여러 데이터베이스 연결을 생성·재사용해 요청을 처리하는 구조 |
| Graceful Shutdown | 새 요청을 막고 진행 작업과 연결을 정리한 뒤 종료하는 방식 |
| Sequelize | Node.js에서 관계형 데이터베이스 모델을 다루는 ORM |
| Transaction | 여러 데이터 변경을 하나의 원자적 작업으로 처리하는 기능 |

## 연결해서 기억하기

MongoDB는 컬렉션 안에 BSON 문서를 저장하고 ObjectId로 식별합니다. Mongoose Schema와 Model은 애플리케이션 규칙과 CRUD를 제공하며, 관계는 포함하거나 참조 후 Populate할 수 있습니다. Express 서버는 연결 수명주기와 오류를 별도로 관리해야 합니다.

## 관련 학습

- [문서형 데이터베이스 기초](01-document-database-foundations.md)
- [Mongoose Schema와 Model](03-mongoose-schema-and-model.md)
- [문서 관계와 Populate](05-document-relations-and-populate.md)
- [ODM과 ORM 선택](07-odm-and-orm-selection.md)
