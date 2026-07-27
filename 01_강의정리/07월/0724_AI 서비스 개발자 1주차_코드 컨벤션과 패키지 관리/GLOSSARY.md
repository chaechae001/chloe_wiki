# 용어집

이번 회차의 핵심 용어를 코드 작성, 문서화와 구조, 품질 자동화, 패키지 관리, 환경 재현으로 나누어 정리했습니다. 각 용어의 관련 글을 따라가면 읽기 쉬운 코드에서 재현 가능한 프로젝트까지 자연스럽게 복습할 수 있습니다.

## 읽기 쉬운 코드와 함수 설계

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 코드 컨벤션(Code Convention) | 들여쓰기·공백·이름·import 배치처럼 프로젝트에서 코드를 어떤 모습으로 쓸지 정한 공통 규칙 | [01](01-code-conventions-and-readable-python.md) | 가독성, 일관성 |
| PEP 8 | 파이썬 코드를 일관되고 읽기 쉽게 작성할 때 출발점으로 삼을 수 있는 대표적인 스타일 기준 | [01](01-code-conventions-and-readable-python.md) | 들여쓰기, 줄 길이 |
| 가독성(Readability) | 코드를 읽는 사람이 구조와 의도를 빠르고 정확하게 파악할 수 있는 정도 | [01](01-code-conventions-and-readable-python.md) | 유지보수, 코드 리뷰 |
| 이름 규칙(Naming Convention) | 변수는 값, 함수는 행동, 클래스는 대상처럼 역할이 드러나도록 식별자를 정하는 기준 | [02](02-naming-functions-and-type-hints.md) | 불리언 이름, 상수 |
| 단일 책임(Single Responsibility) | 함수 하나가 변경 이유가 같은 한 가지 역할에 집중하도록 설계하는 원칙 | [02](02-naming-functions-and-type-hints.md) | 함수 분리, 테스트 |
| 반환값(Return Value) | 함수가 처리 결과를 호출한 코드에 돌려주는 값. 화면에 표시만 하는 `print`와 역할이 다름 | [02](02-naming-functions-and-type-hints.md) | `return`, 재사용 |
| 타입 힌트(Type Hint) | 함수 인자와 반환값이 어떤 타입일 것으로 기대하는지 코드에 표시하는 설명 정보 | [02](02-naming-functions-and-type-hints.md) | 정적 정보, 실행 검증 |
| `Optional[T]` | 값이 `T` 타입이거나 `None`일 수 있음을 나타내는 타입 표기 | [02](02-naming-functions-and-type-hints.md) | `None`, 분기 처리 |

## 문서화·예외 처리와 프로젝트 구조

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 주석(Comment) | 코드가 이미 보여 주는 동작보다 그 구현을 선택한 이유나 제약을 짧게 설명하는 기록 | [03](03-documentation-errors-and-project-structure.md) | 코드 의도, 유지보수 |
| docstring | 함수·클래스·모듈의 목적, 입력, 반환값, 예외처럼 사용자가 알아야 할 계약을 기록하는 문자열 | [03](03-documentation-errors-and-project-structure.md) | 타입 힌트, 사용 계약 |
| 예외 처리(Exception Handling) | 예상 가능한 실패를 구분해 복구하거나 호출자에게 명확한 맥락과 함께 전달하는 과정 | [03](03-documentation-errors-and-project-structure.md) | 실패 경계, 오류 메시지 |
| 구체적 예외 | `FileNotFoundError`처럼 실패 원인을 특정해 서로 다른 대응을 가능하게 하는 예외 타입 | [03](03-documentation-errors-and-project-structure.md) | `except`, 원인 보존 |
| 복구(Recovery) | 실패가 발생해도 의미가 왜곡되지 않는 대체 흐름으로 작업을 계속하는 처리 | [03](03-documentation-errors-and-project-structure.md) | 기본값, 오류 숨기기 |
| 프로젝트 구조(Project Structure) | 핵심 기능·설정·실행 코드·테스트를 역할과 변경 이유에 따라 찾기 쉽게 배치한 구성 | [03](03-documentation-errors-and-project-structure.md) | 모듈화, 책임 분리 |

## 품질 자동화와 코드 리뷰

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| 포매터(Formatter) | 실행 의미는 유지하면서 공백·들여쓰기·줄바꿈 같은 코드 표현을 일정하게 정리하는 도구 | [04](04-formatters-linters-and-code-review.md) | Black, 스타일 |
| 린터(Linter) | 코드를 실행하기 전에 규칙 위반, 사용하지 않는 요소, 잠재 문제를 정적으로 찾아 주는 도구 | [04](04-formatters-linters-and-code-review.md) | Ruff, 정적 분석 |
| Black | 파이썬 코드의 공백과 줄바꿈 등 표현 형식을 일관되게 맞추는 포매터 | [04](04-formatters-linters-and-code-review.md) | 포매터, PEP 8 |
| Ruff | 정적 규칙 위반과 잠재 문제를 진단하고, 안전한 일부 항목을 자동 수정할 수 있는 린터 | [04](04-formatters-linters-and-code-review.md) | `ruff check`, 자동 수정 |
| isort | 표준 라이브러리·외부 패키지·프로젝트 모듈의 import를 정해진 그룹과 순서로 정리하는 도구 | [04](04-formatters-linters-and-code-review.md) | import 그룹, 자동화 |
| AI 코드 리뷰 | AI가 만든 코드의 요구사항, 경계 입력, 하드코딩, 전역 상태, 함수 책임을 사람이 검증하는 과정 | [04](04-formatters-linters-and-code-review.md) | 기능 테스트, 사람 검토 |

## 패키지와 의존성 관리

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| PyPI | 공개된 파이썬 패키지와 버전 정보를 찾을 수 있는 패키지 저장소 | [05](05-pypi-pip-and-requirements.md) | pip, 패키지 배포 |
| pip | 패키지 요구사항을 해석해 현재 선택된 파이썬 환경에 설치하는 명령행 도구 | [05](05-pypi-pip-and-requirements.md) | PyPI, `python -m pip` |
| 의존성(Dependency) | 프로젝트가 실행되기 위해 필요로 하는 외부 패키지와 그 버전 조건 | [05](05-pypi-pip-and-requirements.md) | 패키지, 버전 제약 |
| 직접 의존성(Direct Dependency) | 프로젝트가 기능을 위해 명시적으로 선택하고 선언한 패키지 | [05](05-pypi-pip-and-requirements.md) | 전이 의존성, 선언 |
| 전이 의존성(Transitive Dependency) | 직접 의존성이 다시 요구해 의존성 해결 과정에서 함께 설치되는 하위 패키지 | [05](05-pypi-pip-and-requirements.md) | 의존성 트리, 잠금 파일 |
| 버전 제약(Version Constraint) | `==`, `>=`, `<` 같은 연산자로 설치를 허용할 패키지 버전의 범위를 표현한 규칙 | [05](05-pypi-pip-and-requirements.md) | 호환성, 의존성 충돌 |
| `requirements.txt` | pip가 읽을 수 있도록 필요한 패키지와 버전 조건을 줄 단위로 기록한 설치 요구사항 파일 | [05](05-pypi-pip-and-requirements.md) | `pip freeze`, 설치 목록 |
| 가상환경(Virtual Environment) | 프로젝트별 파이썬 인터프리터와 패키지 설치 위치를 분리해 버전 충돌을 줄이는 환경 경계 | [06](06-virtual-environments-and-uv.md) | 전역 환경, 환경 격리 |

## uv와 재현 가능한 프로젝트 환경

| 용어 | 쉬운 설명 | 관련 글 | 함께 보면 좋은 개념 |
|---|---|---|---|
| `.venv` | 프로젝트 전용 가상환경의 실행 파일과 설치된 패키지가 놓이는 로컬 디렉터리 | [06](06-virtual-environments-and-uv.md) | 가상환경, 환경 재생성 |
| `uv venv` | 프로젝트에서 사용할 가상환경을 생성하는 명령 | [06](06-virtual-environments-and-uv.md) | `.venv`, 활성화 |
| `uv pip` | pip와 유사한 명령 방식으로 가상환경의 패키지를 설치하고 확인하는 인터페이스 | [06](06-virtual-environments-and-uv.md) | `uv pip install`, `uv pip check` |
| `uv run` | 현재 셸의 활성화 여부에만 의존하지 않고 프로젝트 환경에서 명령을 실행하는 기능 | [06](06-virtual-environments-and-uv.md) | 인터프리터, 테스트 실행 |
| `pyproject.toml` | 프로젝트 정보, 지원 파이썬 범위, 직접 의존성과 도구 설정을 사람이 관리하는 선언 파일 | [07](07-pyproject-lockfiles-and-reproducibility.md) | 직접 의존성, 프로젝트 설정 |
| `uv.lock` | 의존성 조건을 해석해 선택된 직접·전이 패키지의 정확한 버전 조합을 기록하는 잠금 파일 | [07](07-pyproject-lockfiles-and-reproducibility.md) | lockfile, 재현 가능성 |
| `uv add` | 직접 의존성을 프로젝트 선언에 추가하고 잠금 상태 갱신 흐름으로 연결하는 명령 | [07](07-pyproject-lockfiles-and-reproducibility.md) | `pyproject.toml`, `uv.lock` |
| `uv sync` | 로컬 가상환경의 설치 상태를 프로젝트의 선언과 잠금 정보에 맞추는 명령 | [07](07-pyproject-lockfiles-and-reproducibility.md) | 환경 동기화, 기능 검증 |
