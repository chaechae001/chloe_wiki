# 문서 관계와 Populate

Mongoose의 Populate는 참조 ID를 관련 문서로 바꿔 주지만 데이터 모델과 쿼리 비용을 대신 결정해 주지는 않습니다.

**핵심 키워드:** reference, ObjectId, ref, populate, N+1

## 핵심 내용

- 참조 관계는 다른 문서의 ObjectId를 필드에 저장합니다.
- Schema의 `ref`는 어떤 Model을 Populate할지 알려 줍니다.
- Populate는 애플리케이션 수준에서 추가 조회를 조합하는 기능이며 데이터베이스 JOIN과 완전히 같지 않습니다.
- 필요한 필드만 선택하고 목록 크기를 제한해 과도한 조회를 피합니다.
- 삭제·권한·일관성 정책은 참조만 선언한다고 자동으로 해결되지 않습니다.

## 참조 Schema

```javascript
const commentSchema = new mongoose.Schema({
  message: { type: String, required: true },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true,
  },
});
```

## Populate 사용

```javascript
const comments = await Comment.find({ postId })
  .populate("author", "name profileImage")
  .limit(50);
```

- **목적:** 댓글 목록의 작성자 ID를 공개 가능한 작성자 필드로 확장합니다.
- **흐름:** 댓글 조회 → 참조 ID 수집 → 관련 사용자 조회 → 결과 결합입니다.
- **결과:** `author` 위치에 선택한 사용자 정보가 포함됩니다.
- **실무 포인트:** 비밀번호 해시, 내부 권한처럼 민감한 필드를 Populate 결과에 포함하지 않습니다.

## 포함과 Populate 비교

| 방식 | 장점 | 주의점 |
|---|---|---|
| 문서 포함 | 한 번에 읽기 쉬움 | 중복과 큰 문서 |
| 참조 + Populate | 독립 데이터 재사용 | 추가 조회와 누락된 참조 |

작성자 표시 이름이 자주 바뀌고 여러 콘텐츠가 공유한다면 참조가 유리할 수 있습니다. 주문 당시 상품 가격처럼 과거 상태를 보존해야 하는 값은 포함된 스냅샷이 더 적합할 수 있습니다.

## 관계 무결성

참조 대상 삭제 시 댓글을 함께 삭제할지, 작성자 없음으로 남길지, 삭제를 막을지 정책을 정합니다. MongoDB와 Mongoose는 관계형 외래 키처럼 모든 정책을 자동 강제하지 않으므로 서비스 로직, 트랜잭션 또는 정리 작업을 설계합니다.

## 실습

1. 게시글이 작성자를 참조하도록 Schema를 작성하세요.
2. 작성자의 이름만 Populate하도록 조회하세요.
3. 사용자 삭제 시 연관 게시글 처리 정책을 세 가지 비교하세요.

<details>
<summary>답</summary>

```javascript
const postSchema = new mongoose.Schema({
  title: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
});

const post = await Post.findById(id).populate("author", "name");
```

연쇄 삭제, 익명화·참조 제거, 삭제 차단 정책을 서비스 요구와 감사 필요성에 맞춰 선택할 수 있습니다.

</details>

## 더 알아보기

- [Mongoose CRUD와 Query](04-mongoose-crud-and-queries.md)
- [Express와 Mongoose 연결 관리](06-express-mongoose-integration.md)

## 체크리스트

- [ ] 포함과 참조의 수명주기를 비교한다.
- [ ] ref가 가리키는 Model을 확인한다.
- [ ] Populate 필드를 최소화한다.
- [ ] 목록 크기와 쿼리 수를 관찰한다.
- [ ] 참조 대상 삭제 정책을 정의한다.

## 복습 질문 및 답변

### Q1. Populate는 MongoDB의 외래 키 제약을 만들어 주나요?

<details>
<summary>답</summary>

아닙니다. 참조 문서를 조회해 결과를 결합하는 ODM 기능이며 대상 존재와 삭제 정책을 관계형 외래 키처럼 자동 강제하지 않습니다.

</details>

### Q2. 모든 참조를 항상 Populate하면 좋은가요?

<details>
<summary>답</summary>

아닙니다. 필요하지 않은 데이터와 추가 조회 비용이 생기므로 API 응답에 필요한 관계와 필드만 선택합니다.

</details>

### Q3. 주문 상품 가격을 현재 상품 문서 참조만으로 표현하면 어떤 문제가 있나요?

<details>
<summary>답</summary>

상품 가격이 바뀌면 과거 주문 당시 가격을 잃을 수 있습니다. 거래 시점의 가격과 이름은 주문 문서에 스냅샷으로 포함하는 방식을 고려합니다.

</details>

## 요약

Populate는 ObjectId 참조를 편리하게 조회하지만 모델링과 무결성 정책은 애플리케이션의 책임입니다. 접근 패턴, 공개 필드와 관계 수명주기를 기준으로 사용하세요.
