# DDIA Chapter 1~2 Overview
참고: Designing Data-Intensive Applications — Martin Kleppmann (O'Reilly, 2017)

---

## Chapter 1 — Reliable, Scalable, and Maintainable Applications

**이 챕터의 핵심 질문: 좋은 데이터 시스템이란 무엇인가?**

세 개의 성질로 답한다.

**Reliability** — 시스템이 잘못된 상황에서도 올바르게 동작하는 것. 중요한 구분은 fault와 failure다. fault는 구성요소 하나가 오작동하는 것이고, failure는 전체 시스템이 서비스 불가 상태가 되는 것이다. 목표는 fault를 없애는 게 아니라 fault가 failure로 번지지 않도록 설계하는 것이다. 장애 원인은 하드웨어 fault, 소프트웨어 fault, 휴먼 에러 세 가지인데, 현실에서는 설정 오류 같은 휴먼 에러가 하드웨어 장애보다 더 많은 다운타임을 만든다.

**Scalability** — "이 시스템은 scalable하다"는 말 자체가 무의미하다. 항상 질문 형태로 논의해야 한다. "부하가 이렇게 증가했을 때 어떤 선택지가 있는가?" 부하를 측정하는 단위가 load parameter이고 시스템마다 다르다. Twitter 예시에서 핵심 load parameter가 초당 트윗 수가 아니라 팔로워 수의 분포였다는 게 이 챕터의 핵심 인사이트다. 성능 측정에서는 평균 대신 percentile을 써야 한다. p99가 느리다는 건 가장 많은 데이터를 가진 고가치 유저가 느린 경험을 한다는 의미다.

**Maintainability** — Operability(운영팀이 시스템 상태를 파악하고 운영할 수 있어야 함), Simplicity(새 엔지니어가 코드를 이해할 수 있어야 함 — 추상화가 핵심 도구), Evolvability(요구사항 변화를 쉽게 반영할 수 있어야 함). 소프트웨어 비용의 대부분이 초기 개발이 아니라 유지보수에 있다는 전제에서 출발한다.

**챕터 1의 흐름**: 좋은 시스템을 정의하고, 각 성질을 어떻게 측정하고 달성하는가를 다룬다. 이후 모든 챕터에서 나오는 설계 결정의 판단 기준이 된다.

---

## Chapter 2 — Data Models and Query Languages

**이 챕터의 핵심 질문: 데이터를 어떻게 표현하고 저장할 것인가?**

**Relational model** — 테이블과 행. Edgar Codd 1970년. 핵심 아이디어는 구현 세부사항(어떤 인덱스를 쓸지, 어떤 순서로 스캔할지)을 query optimizer에게 넘기고 개발자는 "무엇을 원하는가"만 선언한다는 것. many-to-many 관계에 강하고 조인이 자연스럽다.

**Document model** — JSON/BSON. 연관 데이터가 한 문서에 모여 있어서 one-to-many 구조에서 조회가 빠르다(storage locality). schema-on-read라 유연하다. 하지만 many-to-many가 생기는 순간 조인을 애플리케이션 코드에서 직접 해야 하고, 복잡성이 DB에서 앱으로 이동할 뿐이다.

**Impedance mismatch** — 객체지향 언어의 중첩 구조와 관계형 DB의 테이블 사이의 불일치. ORM이 boilerplate를 줄여주지만 근본적 차이를 없애지는 못한다.

**Graph model** — 노드와 엣지. 관계 자체가 핵심 데이터인 경우(소셜 그래프, 도로 네트워크)에 가장 자연스럽다. 어떤 노드도 어떤 노드와도 연결될 수 있어서 관계 유형에 제한이 없다.

**Schema-on-write vs read** — 언제 스키마를 강제하느냐의 차이. schema-on-write는 DB가 쓰기 시점에 검증(관계형), schema-on-read는 읽는 코드가 구조를 해석(문서). 스키마 변경 시 관계형은 ALTER TABLE이 필요하고 대용량 테이블에서 비용이 크다. 문서는 새 구조로 새 문서를 쓰기 시작하면 된다.

**Declarative vs Imperative** — SQL(선언형)은 "무엇을"만 선언하고 query optimizer가 실행 계획을 결정한다. 명령형은 "어떻게"를 단계별로 지시한다. 선언형이 병렬 실행에 유리하고 인덱스 변경에 덜 민감하다.

**챕터 2의 흐름**: 세 가지 모델(관계형/문서/그래프)을 비교하면서, 데이터의 관계 복잡도에 따라 적합한 모델이 달라진다는 것을 보여준다. 하나의 정답이 없고 트레이드오프다.

---

## Chapter 1~2 연결

챕터 1은 "왜 이런 선택을 해야 하는가"의 판단 기준이다. 챕터 2는 그 기준 위에서 "어떤 데이터 모델을 고를 것인가"를 다룬다.

챕터 2를 마치면 "MongoDB는 문서 DB다"가 아니라 "one-to-many 구조에서 locality 이점이 있고, many-to-many가 생기면 복잡성이 앱으로 이동하며, 이 트레이드오프를 reliability/scalability/maintainability 관점에서 판단해야 한다"고 말할 수 있게 된다.
