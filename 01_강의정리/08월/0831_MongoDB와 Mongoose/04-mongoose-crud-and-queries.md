# Mongoose CRUD와 Query

Mongoose CRUD는 메서드를 외우는 것보다 필터, 업데이트, 반환값과 검증 옵션을 정확히 이해하는 것이 중요합니다.

**핵심 키워드:** create, find, filter, update, delete

## 핵심 내용

- `create`는 문서를 생성하고 검증 후 저장합니다.
- `find`, `findOne`, `findById`는 결과 개수와 식별 방식이 다릅니다.
- 쿼리 필터에 사용자 입력을 그대로 전달하지 않고 허용 필드를 구성합니다.
- 업데이트 메서드는 기본 반환값과 검증 실행 여부를 명시적으로 확인합니다.
- 삭제는 권한, 연관 데이터와 복구 정책을 확인한 뒤 수행합니다.

## 생성과 조회

```javascript
const article = await Article.create({
  title: "Mongoose Query",
  status: "draft",
});

const drafts = await Article.find({ status: "draft" })
  .sort({ createdAt: -1 })
  .limit(20);

const one = await Article.findById(article._id);
```

- **목적:** 새 문서를 만들고 최신 초안 목록과 상세를 조회합니다.
- **흐름:** 생성 검증 → 저장 → 필터 → 정렬 → 제한 → ID 조회입니다.
- **결과:** Document 또는 Document 배열을 얻습니다.
- **실무 포인트:** 페이지네이션 없는 대량 `find`는 메모리와 응답 시간을 악화시킬 수 있습니다.

## 조회 메서드 비교

| 메서드 | 일반 반환 |
|---|---|
| `find` | 배열, 결과가 없으면 빈 배열 |
| `findOne` | 한 문서 또는 null |
| `findById` | _id로 찾은 한 문서 또는 null |

## 필터 연산자

```javascript
const selected = await Article.find({
  status: { $in: ["draft", "published"] },
  createdAt: { $gte: startDate },
});
```

`$in`은 필드 값이 후보 목록 중 하나인지 검사합니다. 클라이언트가 전달한 객체를 필터로 그대로 사용하면 의도하지 않은 연산자나 필드가 들어올 수 있으므로 서버가 허용된 필터를 조립합니다.

## 수정과 삭제

```javascript
const updated = await Article.findByIdAndUpdate(
  articleId,
  { $set: { status: "published" } },
  { new: true, runValidators: true }
);

const deleted = await Article.findByIdAndDelete(articleId);
```

`new: true`는 갱신 후 문서를 반환하고 `runValidators: true`는 업데이트에서도 Schema 검증을 적용하도록 요청합니다. 메서드와 버전에 따른 기본 동작은 문서로 확인합니다.

## 실습

1. 상태가 draft 또는 review인 문서를 최신순 10개 조회하세요.
2. 제목을 수정하고 갱신된 문서를 반환받으세요.
3. null 결과와 형식이 잘못된 ID 오류를 API에서 구분하세요.

<details>
<summary>답</summary>

```javascript
const docs = await Article.find({ status: { $in: ["draft", "review"] } })
  .sort({ createdAt: -1 })
  .limit(10);

const updated = await Article.findByIdAndUpdate(
  id,
  { $set: { title } },
  { new: true, runValidators: true }
);
```

</details>

## 더 알아보기

- [Mongoose Schema와 Model](03-mongoose-schema-and-model.md)
- [문서 관계와 Populate](05-document-relations-and-populate.md)

## 체크리스트

- [ ] 조회 메서드의 반환 형태를 구분한다.
- [ ] 사용자 입력으로 필터를 직접 만들지 않는다.
- [ ] 목록 조회에 제한과 정렬을 둔다.
- [ ] 업데이트 검증과 반환 옵션을 확인한다.
- [ ] 삭제 전 권한과 연관 데이터를 확인한다.

## 복습 질문 및 답변

### Q1. `find` 결과가 없으면 null인가요?

<details>
<summary>답</summary>

일반적으로 빈 배열을 반환합니다. 한 문서를 찾는 `findOne`이나 `findById`는 없을 때 null을 반환할 수 있습니다.

</details>

### Q2. 업데이트에서도 Schema 검증이 항상 자동 실행되나요?

<details>
<summary>답</summary>

메서드에 따라 옵션이 필요할 수 있습니다. 사용하는 업데이트 방식의 검증 동작을 확인하고 `runValidators` 같은 옵션을 명시합니다.

</details>

### Q3. 클라이언트가 보낸 query 객체를 MongoDB 필터로 그대로 쓰면 왜 위험한가요?

<details>
<summary>답</summary>

허용하지 않은 필드나 연산자가 들어가 데이터 노출, 비싼 쿼리 또는 예기치 않은 조건을 만들 수 있기 때문입니다.

</details>

## 요약

Mongoose CRUD는 반환값과 옵션까지 API 계약의 일부입니다. 허용 필터, 페이지 제한, 검증과 권한을 명시적으로 적용해야 안전한 데이터 접근 계층이 됩니다.
