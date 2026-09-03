# 용어집

| 용어 | 설명 |
|---|---|
| Configuration | 모델 구조와 동작 기본값을 기록한 설정 |
| Checkpoint | 특정 시점의 모델 파라미터와 관련 상태 저장본 |
| Weight | 학습으로 갱신된 텐서 값 |
| safetensors | 텐서 저장을 위해 설계된 안전한 역직렬화 포맷 |
| Shard | 큰 checkpoint를 나눈 개별 조각 파일 |
| Index file | 파라미터가 어느 shard에 있는지 기록한 매핑 파일 |
| Auto class | config를 보고 알맞은 구현 클래스를 선택하는 인터페이스 |
| Task head | 분류·생성 등 특정 과업 출력을 만드는 모델의 마지막 부분 |
| `state_dict` | 이름과 텐서를 대응시킨 모델 저장 상태 사전 |
| Parameter | 최적화로 학습될 수 있는 모델 텐서 |
| Buffer | 모델 상태로 저장·이동되지만 보통 gradient로 학습하지 않는 값 |
| `numel` | 텐서가 가진 전체 원소 수 |
| Vocabulary | 토큰과 정수 ID의 대응 집합 |
| Subword | 단어보다 작은 재사용 가능한 토큰 단위 |
| Special token | 문장 시작·끝·패딩·미등록 등을 표시하는 제어 토큰 |
| Fast tokenizer | Rust 기반 Tokenizers 구현을 사용하는 빠른 토크나이저 |
| Offset mapping | 각 토큰이 원문에서 차지하는 문자 범위 |
| Padding | batch 길이를 맞추기 위해 특수 토큰으로 채우는 처리 |
| Truncation | 최대 길이를 넘는 토큰을 자르는 처리 |
| Attention mask | 실제 토큰과 padding 위치 등을 구분하는 mask |
| Chat template | 대화 메시지를 모델별 제어 토큰 형식으로 바꾸는 Jinja 템플릿 |
| Generation prompt | 다음 assistant 응답의 시작을 알리는 템플릿 표식 |

## 함께 보기

- [가중치 포맷과 샤딩](02-weight-formats-and-sharding.md)
- [토큰화와 offset mapping](05-tokenization-and-offsets.md)
- [채팅 템플릿과 대화 입력](07-chat-templates.md)
