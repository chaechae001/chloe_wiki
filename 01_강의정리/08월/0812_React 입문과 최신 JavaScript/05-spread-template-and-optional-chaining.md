# 전개 구문과 안전한 값 접근

전개 구문, 템플릿 리터럴, 옵셔널 체이닝은 데이터를 복사·조합하고 표시하며 중첩 값에 안전하게 접근하는 데 쓰입니다.

## 핵심 키워드

`spread syntax` · `template literal` · `optional chaining` · `copy` · `short property`

## 핵심 요약

- 전개 구문은 배열 요소나 객체 프로퍼티를 펼칩니다.
- 전개를 이용한 복사는 얕은 복사입니다.
- 단축 속성명은 변수명과 프로퍼티명이 같을 때 중복을 줄입니다.
- 옵셔널 체이닝은 중간 값이 nullish이면 안전하게 `undefined`를 반환합니다.

## 1. 배열과 객체 전개

```javascript
const base = ["A", "B"];
const extended = [...base, "C"];

const user = { id: 1, name: "새싹" };
const updated = { ...user, name: "푸른새싹" };
```

객체에서는 뒤에 쓴 같은 이름의 프로퍼티가 앞의 값을 덮어씁니다. 중첩 객체는 같은 참조를 공유할 수 있으므로 필요한 깊이까지 복사해야 합니다.

## 2. 단축 속성명과 템플릿 리터럴

```javascript
const title = "React";
const level = "basic";
const course = { title, level };
const label = `${course.title} - ${course.level}`;
```

단축 속성명은 기존 변수로 객체를 만들 때 사용합니다. 템플릿 리터럴은 문자열과 표현식을 읽기 쉽게 조합하고 여러 줄 문자열도 작성할 수 있습니다.

## 3. 옵셔널 체이닝

```javascript
const response = { user: { profile: { nickname: "새싹" } } };
const nickname = response.user?.profile?.nickname;
const city = response.user?.address?.city;

console.log(nickname); // 새싹
console.log(city); // undefined
```

`?.` 앞 값이 `null` 또는 `undefined`일 때만 체인을 멈춥니다. 값이 반드시 있어야 하는 필수 데이터의 오류까지 숨기지 않도록 사용 목적을 구분해야 합니다.

## 대표 코드: 상태 객체 갱신

### 목적

기존 객체를 직접 바꾸지 않고 중첩 프로퍼티를 갱신합니다.

```javascript
const state = {
  user: {
    name: "새싹",
    settings: { theme: "light", alerts: true },
  },
};

const nextState = {
  ...state,
  user: {
    ...state.user,
    settings: {
      ...state.user.settings,
      theme: "dark",
    },
  },
};

console.log(nextState.user.settings.theme); // dark
console.log(state.user.settings.theme); // light
```

### 코드 흐름과 결과

1. 최상위 객체를 복사합니다.
2. 변경 경로의 `user`와 `settings`도 각각 복사합니다.
3. 마지막에 새 theme 값을 덮어씁니다.
4. 이전 객체는 유지되고 새 객체에만 변경이 반영됩니다.

### 실무 연결

React 상태 갱신, API 응답 정규화, 선택적 사용자 정보 표시에서 자주 사용됩니다.

## 직접 해보기

1. 전개 구문의 얕은 복사를 설명하세요.
2. 배열 앞에 새 항목을 추가한 새 배열을 만드세요.
3. 필수 값에 옵셔널 체이닝을 남용할 때의 문제를 설명하세요.

<details>
<summary>정답 보기</summary>

1. 최상위 컨테이너는 새로 만들지만 내부 중첩 객체의 참조는 그대로 공유할 수 있다는 뜻입니다.
2. `const next = [newItem, ...items];`로 만들 수 있습니다.
3. 데이터 계약 위반이 오류 대신 undefined로 조용히 퍼져 원인을 늦게 발견할 수 있습니다.

</details>

## 헷갈리기 쉬운 포인트

| 비교 | 차이 |
|---|---|
| 전개 vs 깊은 복사 | 전개는 한 단계만 복사하며 중첩 참조를 자동 복제하지 않습니다. |
| `?.` vs `&&` 체인 | 옵셔널 체이닝은 nullish만 멈추고 0·빈 문자열 같은 값은 유지합니다. |
| 템플릿 리터럴 vs 문자열 연결 | 템플릿 리터럴은 표현식 삽입과 여러 줄 표현이 더 읽기 쉽습니다. |

## 연결되는 개념

- 구조 분해는 [화살표 함수와 구조 분해](04-arrow-functions-and-destructuring.md)에서 확인할 수 있습니다.
- 상태 갱신은 [컴포넌트와 State로 목록 설계](06-components-and-state.md)에서 이어집니다.
- 프로젝트에서 모듈을 쓰는 법은 [React 프로젝트 환경과 구조](07-react-project-setup.md)에서 다룹니다.

## 셀프 체크

- [ ] 배열과 객체 전개를 사용할 수 있다.
- [ ] 중첩 객체를 필요한 깊이까지 복사할 수 있다.
- [ ] 옵셔널 체이닝의 중단 조건을 설명할 수 있다.

## 복습 질문 및 답변

### Q1. `{ ...user, name: "새 이름" }`에서 name은 어떤 값이 되는가?

<details>
<summary>답</summary>

뒤에 작성된 `"새 이름"`이 앞에서 펼친 기존 name을 덮어씁니다.

</details>

### Q2. 옵셔널 체이닝 결과가 없으면 항상 오류가 발생하는가?

<details>
<summary>답</summary>

중간 값이 nullish이면 오류 대신 `undefined`를 반환합니다.

</details>

### Q3. React 상태에서 중첩 객체까지 복사해야 하는 이유는?

<details>
<summary>답</summary>

변경 경로의 기존 참조를 직접 수정하지 않고 새 참조로 변화가 드러나게 하기 위해서입니다.

</details>

## 한 줄 정리

> 전개 구문은 새 컨테이너를 만들고, 템플릿 리터럴과 옵셔널 체이닝은 값을 읽기 좋고 안전하게 다루도록 돕습니다.
