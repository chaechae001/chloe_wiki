# Mongoose Schema와 Model

Mongoose는 유연한 MongoDB 문서에 애플리케이션 수준의 구조, 검증과 모델 API를 더하는 ODM입니다.

**핵심 키워드:** ODM, Schema, Model, validation, middleware

## 핵심 내용

- ODM은 JavaScript 객체와 MongoDB 문서 사이의 매핑을 돕습니다.
- Schema는 필드 타입, 기본값, 필수 여부, 인덱스와 검증 규칙을 정의합니다.
- Model은 Schema를 기반으로 문서를 생성하고 조회하는 API를 제공합니다.
- Mongoose 검증은 애플리케이션 입력 검증을 보완하지만 모든 보안 검사를 대신하지 않습니다.
- Schema 변경은 기존 문서를 자동으로 완전 변환하지 않으므로 데이터 마이그레이션을 계획합니다.

## Schema 정의

```javascript
import mongoose from "mongoose";

const articleSchema = new mongoose.Schema(
  {
    title: { type: String, required: true, trim: true, maxlength: 120 },
    status: {
      type: String,
      enum: ["draft", "published"],
      default: "draft",
    },
    tags: { type: [String], default: [] },
  },
  { timestamps: true }
);

export const Article = mongoose.model("Article", articleSchema);
```

- **목적:** 게시글 문서가 지켜야 할 구조와 기본 동작을 정의합니다.
- **흐름:** Schema 생성 → 옵션 설정 → Model 생성 → CRUD에서 사용입니다.
- **결과:** 생성과 저장 시 타입 변환과 검증이 적용됩니다.
- **실무 포인트:** `unique: true`는 검증기라기보다 고유 인덱스 생성을 돕는 옵션이므로 중복 키 오류도 처리해야 합니다.

## Schema와 Model 비교

| 개념 | 역할 | 비유 |
|---|---|---|
| Schema | 문서 구조와 규칙 정의 | 설계도 |
| Model | 컬렉션 CRUD 인터페이스 | 설계도로 만든 작업 도구 |
| Document | Model로 생성·조회한 개별 데이터 | 실제 인스턴스 |

## 검증의 층

HTTP 경계에서는 입력 형식과 허용 필드를 빠르게 검증하고, Schema에는 데이터 무결성 규칙을 둡니다. 권한과 비즈니스 정책은 서비스 계층에서 확인합니다. 같은 규칙을 중복하더라도 각 경계가 보호하려는 목적이 다릅니다.

## 인덱스 설계

자주 조회·정렬하는 필드에 인덱스를 고려하지만 쓰기 비용과 저장 공간이 늘어납니다. 실제 쿼리와 실행 계획을 관찰해 필요한 인덱스만 유지합니다.

## 실습

1. 이름, 이메일, 역할을 가진 사용자 Schema를 작성하세요.
2. Schema, Model, Document의 차이를 설명하세요.
3. 기존 문서에 새 필수 필드를 추가할 때 필요한 절차를 적으세요.

<details>
<summary>답</summary>

```javascript
const userSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  email: { type: String, required: true, lowercase: true },
  role: { type: String, enum: ["user", "admin"], default: "user" },
});
```

새 필드는 먼저 선택값이나 기본값으로 배포하고 기존 데이터를 채운 뒤 필수 제약을 강화하는 단계적 마이그레이션을 고려합니다.

</details>

## 더 알아보기

- [Mongoose CRUD와 Query](04-mongoose-crud-and-queries.md)
- [문서 관계와 Populate](05-document-relations-and-populate.md)

## 체크리스트

- [ ] ODM의 역할을 설명한다.
- [ ] Schema와 Model을 구분한다.
- [ ] 필수값과 허용 범위를 정의한다.
- [ ] 고유 인덱스 오류를 별도로 처리한다.
- [ ] Schema 변경 시 마이그레이션을 계획한다.

## 복습 질문 및 답변

### Q1. Mongoose Schema가 있으면 MongoDB가 관계형 DB처럼 고정 스키마가 되나요?

<details>
<summary>답</summary>

Mongoose 애플리케이션 경로에 구조와 검증을 적용하지만 다른 클라이언트의 쓰기까지 자동으로 모두 막는 것은 아닙니다. 필요하면 데이터베이스 검증도 고려합니다.

</details>

### Q2. Model은 문서 하나를 뜻하나요?

<details>
<summary>답</summary>

아닙니다. Model은 특정 컬렉션의 문서를 생성·조회·수정하는 인터페이스이고 개별 결과가 Document입니다.

</details>

### Q3. `unique: true`만으로 사용자 친화적인 중복 검증이 끝나나요?

<details>
<summary>답</summary>

아닙니다. 동시 요청에서도 무결성을 지키는 고유 인덱스가 필요하고 발생한 중복 키 오류를 API 오류로 변환해야 합니다.

</details>

## 요약

Mongoose Schema는 문서 규칙을, Model은 컬렉션 작업 API를 제공합니다. 검증, 인덱스와 마이그레이션을 함께 설계해야 유연한 데이터 구조를 안전하게 운영할 수 있습니다.
