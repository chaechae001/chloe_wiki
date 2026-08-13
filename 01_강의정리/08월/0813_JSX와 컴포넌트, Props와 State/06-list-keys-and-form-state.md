# 목록 key와 입력 상태 설계

목록의 key는 항목의 정체성을 유지하고, 입력 State는 사용자가 보는 값과 애플리케이션 데이터를 동기화합니다.

## 핵심 키워드

`key` · `list rendering` · `controlled component` · `form` · `identity`

## 핵심 요약

- key는 형제 목록에서 항목을 안정적으로 식별합니다.
- 배열 인덱스보다 데이터의 고유 id를 사용하는 것이 안전합니다.
- 제어 입력은 value와 onChange를 State에 연결합니다.
- 목록 추가·삭제는 기존 배열을 바꾸지 않고 새 배열로 갱신합니다.

## 1. 목록과 key

```jsx
function StudentList({ students }) {
  return (
    <ul>
      {students.map((student) => <li key={student.id}>{student.name}</li>)}
    </ul>
  );
}
```

key는 화면에 표시하기 위한 Props가 아니라 React의 비교 과정에 쓰이는 특별한 값입니다. 자식에서 id가 필요하면 별도 Props로 전달합니다.

## 2. 제어 입력

```jsx
const [name, setName] = useState("");

<input value={name} onChange={(event) => setName(event.target.value)} />
```

State가 현재 입력값의 기준이 되고 변경 이벤트가 State를 갱신합니다. 검증, 초기화, 제출 제어가 쉬워집니다.

## 대표 코드: 학생 목록 추가와 삭제

### 목적

제어 입력으로 객체를 추가하고 고유 id를 사용해 목록을 안전하게 갱신합니다.

```jsx
import { useState } from "react";

function StudentManager() {
  const [name, setName] = useState("");
  const [students, setStudents] = useState([]);

  const addStudent = () => {
    const trimmed = name.trim();
    if (!trimmed) return;
    setStudents((previous) => [...previous, { id: crypto.randomUUID(), name: trimmed }]);
    setName("");
  };

  const removeStudent = (id) => {
    setStudents((previous) => previous.filter((student) => student.id !== id));
  };

  return (
    <section>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={addStudent}>추가</button>
      <ul>
        {students.map((student) => (
          <li key={student.id}>
            {student.name} <button onClick={() => removeStudent(student.id)}>삭제</button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

### 코드 흐름과 결과

1. 입력값과 객체 배열을 별도 State로 관리합니다.
2. 추가 시 새 id를 가진 객체와 새 배열을 만듭니다.
3. 렌더링 시 id를 key로 사용합니다.
4. 삭제 시 filter로 대상 id를 제외한 새 배열을 만듭니다.

### 실무 연결

태그 편집, 장바구니, 연락처, 동적 폼처럼 항목 순서와 내부 입력 상태를 보존해야 하는 UI에서 중요합니다.

## 직접 해보기

1. key가 자식 Props로 자동 전달되지 않는 이유를 설명하세요.
2. 학생 이름을 수정하는 불변 갱신 코드를 작성하세요.
3. 인덱스 key가 위험해지는 상황을 설명하세요.

<details>
<summary>정답 보기</summary>

1. key는 React의 목록 비교를 위한 특별한 정보이므로 자식 데이터가 필요하면 별도 id Props를 전달해야 합니다.
2. `setStudents((prev) => prev.map((s) => s.id === id ? { ...s, name } : s));`처럼 작성합니다.
3. 항목 삽입·삭제·정렬로 인덱스가 바뀌면 기존 항목의 UI 상태가 다른 데이터에 연결될 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| key vs id Props | key는 React 내부 비교용이고 id Props는 컴포넌트 로직에서 사용합니다. |
| 제어 vs 비제어 입력 | 제어 입력은 State, 비제어 입력은 DOM이 현재값을 관리합니다. |
| push vs 전개 구문 | push는 기존 배열을 변경하고 전개 구문은 새 배열을 만듭니다. |

## 연결되는 개념

- JSX 목록의 기반은 [JSX 문법과 기본 규칙](01-jsx-syntax-and-rules.md)에서 설명합니다.
- Props 콜백은 [Props와 단방향 데이터 흐름](04-props-and-data-flow.md)에서 확인할 수 있습니다.
- 배열 갱신 원리는 [State와 불변 갱신](05-state-and-immutable-updates.md)에서 이어집니다.

## 셀프 체크

- [ ] 안정적인 key를 선택할 수 있다.
- [ ] 제어 입력을 State와 연결할 수 있다.
- [ ] 배열 State를 불변하게 추가·삭제할 수 있다.

## 복습 질문 및 답변

### Q1. key는 전체 앱에서 유일해야 하는가?

<details>
<summary>답</summary>

같은 형제 목록 안에서 항목을 구분할 수 있으면 됩니다.

</details>

### Q2. 입력을 초기화하려면?

<details>
<summary>답</summary>

제어 입력의 기준이 되는 State를 빈 문자열 등 초기값으로 설정합니다.

</details>

### Q3. 삭제 시 splice보다 filter가 자주 쓰이는 이유는?

<details>
<summary>답</summary>

기존 배열을 직접 변경하지 않고 조건에 맞는 새 배열을 반환하기 때문입니다.

</details>

## 한 줄 정리

> 고유 key와 불변 배열 갱신, 제어 입력을 함께 사용하면 동적인 목록 UI의 정체성과 데이터를 안정적으로 관리할 수 있습니다.
