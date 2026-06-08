# 용어집 (Glossary)

이 강의에서 등장한 핵심 용어를 비전공자 눈높이로 정리했습니다. "관련 글"의 링크를 따라가면 예제와 함께 더 자세히 볼 수 있습니다.

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
| --- | --- | --- | --- |
| 함수(Function) | 특정 기능을 하는 코드 묶음. 이름을 붙여 재사용한다. | [01](posts/01-functions-and-methods.md) | 매개변수, return, 메서드 |
| 매개변수(Parameter) | 함수가 밖에서 값을 받아오는 입력 통로. | [01](posts/01-functions-and-methods.md) | 인자, 기본값 |
| return(반환) | 함수가 계산 결과를 함수 밖으로 돌려주는 것. 없으면 `None`. | [01](posts/01-functions-and-methods.md) | print, None |
| print vs return | print는 화면에 보여주기, return은 값을 전달하기. | [01](posts/01-functions-and-methods.md) | None |
| 내장 함수(Built-in) | 파이썬이 미리 만들어 둔 함수(`len`, `max`, `sum` 등). | [01](posts/01-functions-and-methods.md) | 사용자 정의 함수 |
| 메서드(Method) | 자료 뒤에 점(`.`)을 찍어 그 자료에 작동하는 기능. | [01](posts/01-functions-and-methods.md) | 함수, 클래스 |
| 지역변수(Local) | 함수 안에서만 존재하는 변수. 밖에서 부르면 에러. | [02](posts/02-local-global-variables.md) | scope, NameError |
| 전역변수(Global) | 프로그램 어디서나 읽을 수 있는 변수. | [02](posts/02-local-global-variables.md) | global, 클래스 변수 |
| scope(스코프) | 변수가 보이고 통하는 범위. | [02](posts/02-local-global-variables.md) | 지역/전역변수 |
| Shadowing | 같은 이름의 지역변수가 전역변수를 가리는 현상. | [02](posts/02-local-global-variables.md) | 매개변수, global |
| global | 함수 안에서 전역변수를 수정할 수 있게 하는 선언. | [02](posts/02-local-global-variables.md) | Shadowing |
| f-string | 문자열 앞에 `f`를 붙이고 `{}`에 변수를 넣는 출력 방식. | [03](posts/03-fstring-and-list-comprehension.md) | 포맷팅 |
| List Comprehension | for/if를 대괄호 한 줄로 압축해 리스트를 만드는 문법. | [03](posts/03-fstring-and-list-comprehension.md) | 표현식, 조건문 |
| sort(메서드) | 원본 리스트를 직접 정렬. 반환값은 `None`. | [04](posts/04-sorted-enumerate-lambda.md) | sorted |
| sorted(함수) | 정렬된 복사본을 돌려주고 원본은 유지. | [04](posts/04-sorted-enumerate-lambda.md) | sort, reverse |
| enumerate | 순서 있는 자료에서 인덱스와 값을 함께 꺼내는 함수. | [04](posts/04-sorted-enumerate-lambda.md) | for문, start |
| lambda | 이름 없는 한 줄짜리 익명 함수. | [04](posts/04-sorted-enumerate-lambda.md) | map, def |
| map | 시퀀스의 모든 원소에 함수를 한 번에 적용. | [04](posts/04-sorted-enumerate-lambda.md) | lambda, list 형변환 |
| 모듈(Module) | 함수·자료를 담은 `.py` 파일 묶음. | [05](posts/05-modules-and-packages.md) | import, 패키지 |
| import | 모듈을 불러오는 키워드. | [05](posts/05-modules-and-packages.md) | from import |
| from ~ import | 모듈에서 특정 함수만 가져와 점 없이 쓰는 방식. | [05](posts/05-modules-and-packages.md) | import |
| 패키지(Package) | 모듈을 폴더(디렉터리)로 묶어 관리하는 것. | [05](posts/05-modules-and-packages.md) | 모듈 |
| random / math | 난수 생성 / 수학 연산을 돕는 표준 모듈. | [05](posts/05-modules-and-packages.md) | randrange, pi |
| `if __name__ == "__main__"` | 파일을 직접 실행할 때만 동작시킬 코드를 담는 시작점. | [05](posts/05-modules-and-packages.md) | import |
| 클래스(Class) | 객체를 만드는 설계도(청사진). | [06](posts/06-class-and-oop.md) | 인스턴스, 속성, 메서드 |
| 인스턴스(Instance) | 클래스로 만든 실제 객체. | [06](posts/06-class-and-oop.md) | 클래스 |
| `__init__` | 인스턴스 생성 시 자동 실행되는 초기화 메서드. | [06](posts/06-class-and-oop.md) | self, 속성 |
| self | 메서드 안에서 "그 인스턴스 자신"을 가리키는 이름. | [06](posts/06-class-and-oop.md) | cls, 인스턴스 변수 |
| 속성(Attribute) | 객체가 가지는 데이터(이름·연봉 등). | [06](posts/06-class-and-oop.md) | 메서드 |
| 인스턴스 변수 | 객체마다 독립적으로 가지는 값(`self.name`). | [06](posts/06-class-and-oop.md) | 클래스 변수 |
| 클래스 변수 | 모든 인스턴스가 공유하는 값. | [06](posts/06-class-and-oop.md) | 전역변수 |
| 매직 메서드 | `__str__`처럼 밑줄 두 개가 붙은 특별한 메서드. | [06](posts/06-class-and-oop.md) | `__repr__` |
| `@classmethod` | 첫 인자로 `cls`를 받는, 클래스 관련 메서드. | [06](posts/06-class-and-oop.md) | 대체 생성자 |
| `@staticmethod` | `self`/`cls` 없이 동작하는 독립 유틸 메서드. | [06](posts/06-class-and-oop.md) | classmethod |
| 상속(Inheritance) | 부모 클래스의 기능을 자식이 물려받는 것. | [06](posts/06-class-and-oop.md) | super, 오버라이딩 |
| 오버라이딩(Overriding) | 물려받은 메서드를 자식에서 다시 정의하는 것. | [06](posts/06-class-and-oop.md) | 상속, 다형성 |
| 다형성(Polymorphism) | 같은 메서드 호출이 객체마다 다르게 동작하는 것. | [06](posts/06-class-and-oop.md) | 오버라이딩 |
