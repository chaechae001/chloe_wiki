# 작성자 권한, 댓글 API와 Aggregation

회원과 게시글을 연결하면 화면 표시뿐 아니라 변경 권한, 조회 비용과 삭제 정책까지 함께 설계해야 합니다. 댓글 API는 서버 렌더링 화면에 필요한 부분만 비동기로 갱신하는 데 활용할 수 있습니다.

**핵심 키워드:** authorization, populate, subdocument, fetch, aggregation

## 작성자 연결과 권한

```javascript
const postSchema = new Schema({
  title: { type: String, required: true },
  author: { type: Schema.Types.ObjectId, ref: "User", required: true },
  comments: [commentSchema],
});
```

등록 시 작성자는 본문이 아니라 인증 상태에서 가져옵니다. 수정·삭제 필터에 `_id`와 `author`를 함께 넣으면 검사와 변경 사이의 경쟁 조건을 줄일 수 있습니다.

```javascript
const post = await Post.findOneAndUpdate(
  { _id: req.params.id, author: req.user.id },
  { $set: allowedChanges },
  { new: true, runValidators: true }
);
```

## 댓글 API와 클라이언트 렌더링

`fetch`로 댓글 목록을 읽고 작성할 때 서버는 인증, 글 존재 여부, 본문 길이를 검사합니다. DOM을 문자열 결합으로 만들기보다 `textContent`를 사용해 사용자 입력이 HTML로 실행되지 않게 합니다. 상태 코드는 성공·검증 실패·인증 실패·대상 없음에 맞게 구분합니다.

## 포함과 참조

댓글이 게시글과 함께 조회·삭제되고 크기가 제한적이면 서브도큐먼트가 단순합니다. 댓글이 매우 많거나 독립 검색·권한·수명주기가 필요하면 별도 컬렉션을 검토합니다.

## Aggregation

Aggregation Pipeline은 `$match`, `$group`, `$sort`, `$project` 같은 단계를 연결해 통계를 만듭니다. 가능한 한 초기에 `$match`로 범위를 줄이고, 자주 사용하는 조건과 정렬에 맞는 인덱스를 확인합니다.

## 실습

1. 댓글 작성 API의 검증 순서를 정하세요.
2. 댓글을 포함할지 참조할지 기준을 세우세요.
3. 사용자별 게시글 수 집계 단계를 적으세요.

<details>
<summary>답</summary>

인증 → 입력 검증 → 게시글 확인 → 댓글 저장 → 최소 응답 순서로 처리합니다. 독립 수명주기와 규모가 작으면 포함, 크고 별도 조회가 많으면 참조를 고려합니다. 집계는 조건 필터 후 작성자 기준 그룹화와 정렬로 구성합니다.

</details>

## 체크리스트

- [ ] 작성자를 인증 상태에서 결정한다.
- [ ] 변경 쿼리에 소유권 조건을 포함한다.
- [ ] 댓글 본문을 서버에서 검증한다.
- [ ] 사용자 입력을 안전하게 DOM에 넣는다.
- [ ] 집계 범위와 인덱스를 점검한다.

## 복습 질문 및 답변

### Q1. `populate`가 권한 검사를 대신할 수 있나요?

<details>
<summary>답</summary>

아닙니다. `populate`는 참조 문서를 조회하는 기능이며 작업 허용 여부는 별도로 검사해야 합니다.

</details>

### Q2. 댓글 입력을 `innerHTML`로 넣으면 어떤 위험이 있나요?

<details>
<summary>답</summary>

악성 스크립트가 마크업으로 해석되어 저장형 XSS로 이어질 수 있습니다.

</details>

### Q3. Aggregation에서 `$match`를 앞에 두는 이유는 무엇인가요?

<details>
<summary>답</summary>

이후 단계가 처리할 문서 수를 줄이고 적절한 인덱스를 사용할 가능성을 높이기 위해서입니다.

</details>

## 요약

관계 기능은 데이터 연결만이 아니라 소유권, 입력 안전, 모델링 비용과 집계 성능을 함께 결정합니다.
