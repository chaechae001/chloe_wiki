# 클래스와 객체지향 — 설계도로 객체를 찍어내고, 구조를 세우다

> "팀원"이라는 개념에는 이름·직책·연봉 같은 데이터와, 자기소개·프로젝트 합류 같은 행동이 함께 있습니다. 이 둘을 하나로 묶어 다루는 방법이 클래스입니다.

`클래스` `인스턴스` `__init__` `self` `속성` `메서드` `클래스변수` `인스턴스변수` `상속` `오버라이딩` `다형성`

## 핵심요약

- **클래스**는 객체를 만들기 위한 **설계도(청사진)**, **인스턴스**는 그 설계도로 만든 **실제 객체**다.
- 클래스는 **속성(데이터)** 과 **메서드(행동)** 를 함께 담는다.
- `__init__`은 인스턴스 생성 시 **자동 실행**되어 초기 속성을 설정하는 특별한 메서드이고, `self`는 "그 인스턴스 자신"을 가리킨다.
- **인스턴스 변수**는 객체마다 다른 값, **클래스 변수**는 모든 객체가 공유하는 값이다.
- `__str__`/`__repr__` 같은 **매직 메서드**로 객체를 사람이 읽기 좋게 출력할 수 있다.
- `@classmethod`(첫 인자 `cls`)와 `@staticmethod`(인자 없음)로 메서드의 성격을 구분한다.
- **상속**으로 공통 기능을 물려받고, **오버라이딩**으로 자식이 메서드를 재정의하며, 같은 호출이 객체마다 다르게 동작하는 것이 **다형성**이다.

---

## 개념별 정리

### 클래스와 인스턴스

**1. 정의**
클래스는 설계도, 인스턴스는 그 설계도로 만들어진 실제 물건이다.

**2. 왜 필요한가?**
"팀원"처럼 같은 구조를 가진 대상을 여러 개 만들어야 할 때, 설계도를 한 번 정의하고 필요한 만큼 찍어내면 된다.

**3. 예시**

```python
class TeamMember:
    pass   # 아직 내용이 없는 빈 클래스

member1 = TeamMember()   # 인스턴스 1
member2 = TeamMember()   # 인스턴스 2

print(type(member1))         # <class '__main__.TeamMember'>
print(member1)               # <__main__.TeamMember object at 0x...>
print(member1 is member2)    # False (서로 다른 객체)
```

실행 결과:
```
<class '__main__.TeamMember'>
<__main__.TeamMember object at 0x...>
False
```

**4. 헷갈리기 쉬운 점**
같은 클래스로 만들어도 인스턴스끼리는 서로 다른 객체다(`is` 비교가 `False`). 메모리 주소가 다르기 때문이다.

**5. 한 줄 정리**
클래스는 붕어빵 틀, 인스턴스는 그 틀로 구운 붕어빵 하나하나다.

> 비유: 붕어빵 틀(클래스)이 하나여도 거기서 나온 붕어빵(인스턴스)은 각각 다른 빵이다.

### `__init__`과 self, 속성(Attribute)

**1. 정의**
`__init__`은 인스턴스가 생성될 때 자동으로 실행되는 초기화 메서드다. `self`는 "지금 만들어지는 그 인스턴스 자신"을 가리킨다.

**2. 왜 필요한가?**
객체마다 다른 초기 데이터(이름·직책·연봉)를 만들 때 넣어 주기 위해서다.

**3. 예시**

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name       # 인스턴스 속성 설정
        self.role = role
        self.salary = salary

alice = TeamMember(name="Alice", role="시니어 개발자", salary=6000)
bob   = TeamMember(name="Bob",   role="주니어 기획자", salary=3800)

print(f"{alice.name}의 직책: {alice.role}, 연봉: {alice.salary}만원")
print(f"{bob.name}의 직책: {bob.role}, 연봉: {bob.salary}만원")
```

실행 결과:
```
Alice의 직책: 시니어 개발자, 연봉: 6000만원
Bob의 직책: 주니어 기획자, 연봉: 3800만원
```

**4. 헷갈리기 쉬운 점**
`self`를 "어렵게" 느끼기 쉽지만, "초기화/시작점에서 이 객체에 값을 붙인다"는 **프로세스 관점**으로 보면 부담이 줄어든다. `self.name = name`은 "이 객체의 name 칸에 받은 값을 적어 둔다"는 뜻이다.

**5. 한 줄 정리**
`__init__`은 객체가 태어날 때 속성을 채워 주는 자동 실행 메서드이고, `self`는 그 객체 자신이다.

### 인스턴스 메서드(행동)

**1. 정의**
클래스 안에 정의된 함수로, 첫 매개변수는 항상 `self`다. 그 객체가 할 수 있는 "행동"을 담는다.

**2. 왜 필요한가?**
데이터(속성)뿐 아니라 그 데이터를 다루는 행동까지 한 곳에 묶기 위해서다.

**3. 예시**

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary
        self.projects = []   # 처음엔 빈 리스트

    def introduce(self):
        print(f"안녕하세요! 저는 {self.role} {self.name}입니다.")

    def join_project(self, project_name):
        self.projects.append(project_name)
        print(f"{self.name}이(가) '{project_name}' 프로젝트에 합류했습니다.")

    def get_raise(self, amount):
        self.salary += amount
        print(f"{self.name}의 연봉이 {amount}만원 인상되어 {self.salary}만원이 되었습니다.")

alice = TeamMember("Alice", "시니어 개발자", 6000)
alice.introduce()
alice.join_project("AI 챗봇 개발")
alice.get_raise(500)
```

실행 결과:
```
안녕하세요! 저는 시니어 개발자 Alice입니다.
Alice이(가) 'AI 챗봇 개발' 프로젝트에 합류했습니다.
Alice의 연봉이 500만원 인상되어 6500만원이 되었습니다.
```

**4. 헷갈리기 쉬운 점**
메서드를 정의할 땐 `self`를 적지만, 호출할 땐 `alice.introduce()`처럼 `self`를 넣지 않는다. 파이썬이 알아서 `alice`를 `self`로 넘긴다.

**5. 한 줄 정리**
메서드는 그 객체가 할 수 있는 행동이고, `self`로 자기 속성에 접근한다.

### 클래스 변수 vs 인스턴스 변수

**1. 정의**
인스턴스 변수는 객체마다 독립적인 값(`self.name`), 클래스 변수는 모든 객체가 공유하는 값(클래스 안, `__init__` 바깥에 선언)이다.

**2. 왜 필요한가?**
"개별 객체마다 다른 값"과 "모두가 공유해야 하는 값"을 구분해 관리하기 위해서다. 예: 회사명은 공유(클래스 변수), 이름은 개별(인스턴스 변수).

**3. 예시**

```python
class TeamMember:
    company_name = "TechCorp"     # 클래스 변수: 모두 공유
    total_members = 0             # 클래스 변수: 전체 인원 카운터

    def __init__(self, name, role, salary):
        self.name = name          # 인스턴스 변수: 각자 다름
        self.role = role
        self.salary = salary
        TeamMember.total_members += 1   # 생성될 때마다 +1

    def introduce(self):
        print(f"[{self.company_name}] {self.role} {self.name} (연봉: {self.salary}만원)")

alice = TeamMember("Alice", "시니어 개발자", 6000)
bob   = TeamMember("Bob",   "주니어 기획자", 3800)
carol = TeamMember("Carol", "데이터 분석가", 4500)

print(f"현재 총 팀원 수: {TeamMember.total_members}명")

TeamMember.company_name = "TechCorp Korea"   # 클래스 변수 변경
alice.introduce()   # 모든 인스턴스에 반영됨
```

실행 결과:
```
현재 총 팀원 수: 3명
[TechCorp Korea] 시니어 개발자 Alice (연봉: 6000만원)
```

**4. 헷갈리기 쉬운 점**
주피터에서 셀을 여러 번 실행하면 `total_members`가 계속 누적되어 "팀원 수가 이상하게 늘어나는" 현상이 생긴다. 이는 [전역 상태(scope) 개념](02-local-global-variables.md)과 연결된다 — 클래스 변수는 전역적으로 공유되는 상태다. 또 변수명 오타(`total_member` vs `total_members`)도 흔한 에러 원인이다.

**5. 한 줄 정리**
"공유할 값"은 클래스 변수, "객체마다 다른 값"은 인스턴스 변수다.

### 매직 메서드 — `__str__`, `__repr__`

**1. 정의**
이름 앞뒤에 밑줄 두 개가 붙은 특별한 메서드들. `__str__`은 사람이 읽기 좋은 출력, `__repr__`은 디버깅용 공식 표현을 제공한다.

**2. 왜 필요한가?**
정의하지 않으면 객체를 출력할 때 주소값(`<...object at 0x...>`)만 나와 이해하기 어렵다.

**3. 예시**

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary

    def __str__(self):
        return f"{self.role} {self.name}"

    def __repr__(self):
        return f"TeamMember(name='{self.name}', role='{self.role}', salary={self.salary})"

alice = TeamMember("Alice", "시니어 개발자", 6000)
print(str(alice))    # __str__ 호출
print(repr(alice))   # __repr__ 호출
print(alice)         # print()는 __str__ 호출
```

실행 결과:
```
시니어 개발자 Alice
TeamMember(name='Alice', role='시니어 개발자', salary=6000)
시니어 개발자 Alice
```

**4. 헷갈리기 쉬운 점**
`print(객체)`는 `__str__`을, 디버깅/표현은 `__repr__`을 부른다. 매직 메서드는 종류가 많으니 필요할 때 공식 문서(데이터 모델)를 참고한다.

**5. 한 줄 정리**
매직 메서드를 정의하면 객체를 주소값 대신 의미 있는 문자열로 보여 줄 수 있다.

### 클래스 메서드와 정적 메서드

**1. 정의**
`@classmethod`는 첫 인자로 클래스 자신(`cls`)을 받아 클래스 변수에 접근하거나 대체 생성자로 쓴다. `@staticmethod`는 `self`/`cls` 없이 독립적인 유틸리티 함수처럼 동작한다.

**2. 왜 필요한가?**
"특정 인스턴스가 아니라 클래스 전체와 관련된 기능"이나 "객체 없이도 쓸 수 있는 보조 기능"을 자연스럽게 표현하기 위해서다.

**3. 예시**

```python
class TeamMember:
    company_name = "TechCorp"
    total_members = 0

    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary
        TeamMember.total_members += 1

    @classmethod
    def get_total_members(cls):
        return cls.total_members

    @classmethod
    def from_string(cls, member_string):
        # "Alice-시니어 개발자-6000" → 분해하여 객체 생성 (대체 생성자)
        name, role, salary = member_string.split("-")
        return cls(name, role, int(salary))

    @staticmethod
    def is_valid_salary(salary):
        return 1000 <= salary <= 200000

    def __str__(self):
        return f"{self.role} {self.name} (연봉: {self.salary}만원)"

alice = TeamMember("Alice", "시니어 개발자", 6000)
bob   = TeamMember("Bob",   "주니어 기획자", 3800)
print(f"현재 팀원 수: {TeamMember.get_total_members()}명")

carol = TeamMember.from_string("Carol-데이터 분석가-4500")   # 문자열로 객체 생성
print(carol)
print(f"총 팀원 수: {TeamMember.get_total_members()}명")

print(f"6000만원은 유효한 연봉인가? {TeamMember.is_valid_salary(6000)}")
print(f"500만원은 유효한 연봉인가? {TeamMember.is_valid_salary(500)}")
```

실행 결과:
```
현재 팀원 수: 2명
데이터 분석가 Carol (연봉: 4500만원)
총 팀원 수: 3명
6000만원은 유효한 연봉인가? True
500만원은 유효한 연봉인가? False
```

**4. 헷갈리기 쉬운 점**
`from_string`은 `__init__`이 아닌데도 객체를 만든다 — 이를 "대체 생성자"라 한다. `cls(...)`가 곧 `TeamMember(...)`다. 정적 메서드(`is_valid_salary`)는 객체 없이 `TeamMember.is_valid_salary(6000)`처럼 클래스에 바로 호출한다.

**5. 한 줄 정리**
`@classmethod`는 클래스 전체와 관련된 기능(`cls`), `@staticmethod`는 객체와 무관한 보조 기능이다.

### 상속 · 오버라이딩 · 다형성

**1. 정의**
상속은 부모 클래스의 속성·메서드를 자식이 물려받는 것, 오버라이딩은 물려받은 메서드를 자식이 재정의하는 것, 다형성은 같은 메서드 호출이 객체에 따라 다르게 동작하는 것이다.

**2. 왜 필요한가?**
공통 기능을 부모에 한 번만 작성하고, 부서별 차이만 자식에서 표현하면 중복이 줄고 구조가 명확해진다. 문법보다 **설계(계층 구조)** 이해가 핵심이다.

**3. 예시 (요약)**

```python
class Department:                       # 부모 클래스: 공통 기능
    def __init__(self, team_name, budget):
        self.team_name = team_name
        self.budget = budget
        self.members = []

    def add_member(self, member):
        self.members.append(member)
        print(f"[{self.team_name}] {member.name}님이 합류했습니다.")

    def get_total_salary(self):
        return sum(m.salary for m in self.members)

    def team_meeting(self):             # 자식이 재정의할 수 있는 메서드
        print(f"[{self.team_name}] 정기 팀 회의를 시작합니다.")

class DevTeam(Department):              # 자식 클래스: 개발팀
    def __init__(self, budget, tech_stack):
        super().__init__("개발팀", budget)   # 부모의 __init__ 호출
        self.tech_stack = tech_stack

    def team_meeting(self):             # 오버라이딩: 개발팀만의 회의
        print(f"[{self.team_name}] 스프린트 계획 회의를 시작합니다. (기술 스택: {', '.join(self.tech_stack)})")
```

다형성 — 같은 `team_meeting()` 호출이 팀마다 다르게 동작:

```python
for team in [dev_team, planning_team, sales_team, analytics_team]:
    team.team_meeting()   # 각 팀의 오버라이딩된 메서드가 호출됨
```

실행 결과(발췌):
```
[개발팀] 스프린트 계획 회의를 시작합니다. (기술 스택: Python, React, PostgreSQL, Docker)
[기획팀] 기획 검토 회의를 시작합니다. (현재 기획: AI 기반 고객 추천 서비스)
[영업팀] 영업 현황 회의 (목표: 100000만원, 달성: 75000만원, 75.0%)
[분석팀] 데이터 리뷰 회의 (사용 도구: Python, Tableau, BigQuery, 발행 보고서: 2건)
```

**4. 헷갈리기 쉬운 점**
`super().__init__(...)`을 빼먹으면 부모가 설정해 주던 속성(`team_name`, `members` 등)이 없어 에러가 난다. 오버라이딩은 "같은 이름의 메서드를 자식에서 다시 정의"하는 것이지 삭제가 아니다.

**5. 한 줄 정리**
공통은 부모에서 상속받고, 차이는 자식에서 오버라이딩하며, 같은 호출이 객체마다 다르게 동작하는 게 다형성이다.

> 비유: 부모 클래스는 "전 부서 공통 매뉴얼", 자식 클래스는 "부서별로 일부를 고쳐 쓴 매뉴얼". 모두에게 "회의하세요"라고 말해도(같은 호출) 부서마다 회의 방식이 다른 게 다형성이다.

---

## 코드로 보기 — 한 회사가 만들어지는 과정

```python
# Step 1. 팀원(인스턴스) 생성
alice = TeamMember("Alice", "시니어 백엔드 개발자", 6500)
derek = TeamMember("Derek", "프론트엔드 개발자",   4800)

# Step 2. 팀(부서) 생성
dev_team = DevTeam(budget=50000, tech_stack=["Python", "React", "PostgreSQL", "Docker"])

# Step 3. 팀에 팀원 배정
dev_team.add_member(alice)
dev_team.add_member(derek)

# Step 4. 팀 현황 확인
dev_team.show_members()
print(f"  → 팀 연봉 합계: {dev_team.get_total_salary():,}만원")
```

실행 결과(발췌):
```
[개발팀] Alice님이 합류했습니다.
[개발팀] Derek님이 합류했습니다.

=== 개발팀 구성원 (2명) ===
  1. 시니어 백엔드 개발자: Alice (연봉 6500만원)
  2. 프론트엔드 개발자: Derek (연봉 4800만원)
  → 팀 연봉 합계: 11,300만원
```

**코드목적**
팀원 객체를 만들고, 팀 객체에 배정한 뒤, 팀 단위로 현황과 연봉 합계를 출력한다. 객체들이 서로를 담고 협력하는 전형적인 구조다.

**해석**
`dev_team.members`라는 리스트 안에 `TeamMember` 객체들이 들어가고, `get_total_salary()`는 그 객체들의 `salary`를 모두 더한다. 객체가 객체를 포함하며 역할을 나눠 가진다.

**실무 연결**
이 구조는 실제 서비스 모델링과 닮았다. "사용자–주문–상품", "팀–멤버–프로젝트"처럼 현실의 관계를 클래스로 표현하면, 데이터와 행동을 한 곳에서 일관되게 관리할 수 있다. 또한 `isinstance(객체, 클래스)`로 타입을, `issubclass(자식, 부모)`로 상속 관계를 확인할 수 있다.

---

## 직접 해보기

1. `Book` 클래스를 만들어 `title`, `author`, `price` 속성을 `__init__`으로 설정하고, `introduce()` 메서드로 책 정보를 출력해 보자.
2. `Book`에 클래스 변수 `total_books`를 두고, 객체가 생성될 때마다 1씩 늘어나도록 만들어 보자.
3. `Book`을 상속한 `Ebook` 클래스를 만들어 `file_size` 속성을 추가하고, `introduce()`를 전자책에 맞게 오버라이딩해 보자.

---

## 헷갈리기 쉬운 포인트

- **클래스변수 vs 인스턴스변수**: 모두 공유 vs 객체별 독립. 카운터가 이상하게 누적되면 클래스(전역) 상태를 의심한다.
- **self vs cls**: 인스턴스 메서드는 `self`(그 객체), 클래스 메서드는 `cls`(클래스 자신).
- **__str__ vs __repr__**: 사람용 출력 vs 디버깅용 표현. `print()`는 `__str__`을 부른다.
- **super() 호출**: 자식 `__init__`에서 `super().__init__()`을 빼면 부모 속성이 설정되지 않는다.

---

## 연결되는 개념

- 이전 글: [지역변수 vs 전역변수](02-local-global-variables.md) — 클래스 변수는 "공유 상태"라는 점에서 전역 개념과 통한다.
- 함께 보면 좋은 글: [함수와 메서드](01-functions-and-methods.md) — 메서드는 클래스 안의 함수다.
- 함께 보면 좋은 글: [모듈과 패키지](05-modules-and-packages.md) — 클래스를 모듈로 분리해 재사용한다.
- 더 찾아볼 키워드: 캡슐화, `property`, 추상 클래스, 다중 상속, MRO

---

## 셀프 체크

- [ ] 클래스와 인스턴스의 관계를 비유로 설명할 수 있다.
- [ ] `__init__`과 `self`의 역할을 안다.
- [ ] 클래스 변수와 인스턴스 변수를 구분할 수 있다.
- [ ] `__str__`을 정의해 객체 출력을 바꿀 수 있다.
- [ ] `@classmethod`와 `@staticmethod`의 차이를 안다.
- [ ] 상속·오버라이딩·다형성을 예로 설명할 수 있다.

**복습 질문 및 답변**

- (기본) 클래스와 인스턴스는 어떤 관계인가?
  → 클래스는 설계도, 인스턴스는 그 설계도로 만든 실제 객체다(붕어빵 틀과 붕어빵).
- (이해 확인) `self.name = name`은 무슨 의미인가?
  → 지금 만들어지는 이 객체의 `name` 속성에 전달받은 값을 저장한다는 뜻이다.
- (응용) 모든 부서가 "회의를 한다"는 공통 행동을 가지되 방식이 다르다. 어떻게 설계하나?
  → 부모 클래스에 `team_meeting()`을 정의하고, 각 부서 자식 클래스에서 오버라이딩한다(다형성).

---

## 한 줄 정리

> 클래스는 속성(데이터)과 메서드(행동)를 묶은 설계도이고, `__init__`/`self`로 객체를 초기화하며, 클래스/인스턴스 변수·매직 메서드·클래스/정적 메서드·상속/오버라이딩/다형성으로 현실의 구조를 코드로 설계한다.
