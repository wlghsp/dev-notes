# Refactoring (2nd Edition) Glossary 진도표

참고: Refactoring: Improving the Design of Existing Code, 2nd Edition — Martin Fowler with Kent Beck (Addison-Wesley, 2019). PDF는 assets/에 원본 보관.

전략: 1~5장은 리팩토링이라는 행위 자체의 원칙(왜, 언제, 어떻게 안전하게)을 잡는 파트. 6~12장은 개별 기법 카탈로그 — 기법 하나하나가 독립된 레시피라서, 책 전체를 순서대로 읽기보다 필요할 때 찾아 쓰는 참고서에 가깝다. 그래서 카탈로그 파트는 챕터 순서대로 진행하되, 지호님이 실무에서 마주친 기법을 먼저 짚어도 무방하다.

각 기법 옆 괄호 숫자는 원서 페이지 번호(2nd Edition 기준).

---

## Chapter 1 — Refactoring: A First Example

리팩토링이 실제로 어떤 흐름으로 진행되는지 하나의 예제로 보여주는 챕터. 원칙은 다음 장에서.

- [x] refactoring-first-example.md — 이 챕터의 예제가 보여주는 핵심 흐름: 작은 단계 + 매 단계 테스트. Extract Function, Move Function 등 이후 카탈로그 기법이 실전에서 어떻게 연쇄적으로 쓰이는지의 축소판
- [x] ch01-refactoring-first-example.md — 챕터 1 종합 문서

---

## Chapter 2 — Principles in Refactoring

리팩토링이라는 용어 자체의 정의와, 왜/언제 하는가에 대한 원칙.

- [x] refactoring-definition.md — 리팩토링의 정의. 명사(구조 개선 기법)와 동사(그 기법을 적용하는 행위)로서의 의미 차이
- [x] two-hats.md — 두 개의 모자. 기능 추가와 리팩토링을 동시에 하지 않고 모드를 전환하며 작업하는 원칙
- [x] rule-of-three.md — 3의 법칙. 언제 리팩토링(추상화)할 시점인지 판단하는 경험칙
- [x] preparatory-refactoring.md — 준비 리팩토링. 기능 추가 직전이 리팩토링하기 가장 좋은 시점이라는 원칙
- [x] comprehension-refactoring.md — 이해를 위한 리팩토링. 코드를 이해하는 과정 자체를 코드에 새겨 넣는 습관
- [x] litter-pickup-refactoring.md — 쓰레기 줍기 리팩토링. 나쁘게 짜인 코드를 지날 때마다 조금씩 개선하는 방식
- [x] planned-vs-opportunistic-refactoring.md — 계획된 리팩토링과 기회주의적 리팩토링의 구분
- [x] when-not-to-refactor.md — 리팩토링이 필요 없거나 오히려 다시 짜는 게 나은 경우
- [x] problems-with-refactoring.md — 코드 오너십, 브랜치/CI, 테스팅, 레거시 코드, 데이터베이스(parallel change) 관련 현실적 장애물
- [x] design-stamina-hypothesis.md — 설계 체력 가설과 YAGNI
- [x] refactoring-and-the-wider-process.md — 자가 검증 코드·CI·리팩토링의 시너지
- [x] refactoring-and-performance.md — 리팩토링과 성능(64). 리팩토링이 성능을 희생시킨다는 우려에 대한 반박과 실제 트레이드오프
- [x] ch02-principles-in-refactoring.md — 챕터 2 종합 문서

---

## Chapter 3 — Bad Smells in Code

리팩토링이 필요하다는 신호들. 지호님이 이미 짚은 "domain 객체를 전 영역에서 쓰는" 문제도 이 챕터의 냄새 중 하나(Divergent Change / Shotgun Surgery 계열)와 연결된다.

- [ ] code-smell.md — 코드 스멜이라는 개념 자체. 정량적 기준이 아니라 경험적 직관으로 잡아내는 문제 신호
- [ ] divergent-change.md — 산탄총 수술의 반대 개념. 한 모듈이 여러 이유로 자꾸 바뀌는 문제
- [ ] shotgun-surgery.md — 산탄총 수술. 변경 하나가 여러 모듈에 흩어진 수정을 요구하는 문제 — domain 객체를 전 영역에서 쓰던 지호님 사례와 직결
- [ ] feature-envy.md — 기능 편애(77). 한 모듈이 다른 모듈의 데이터를 지나치게 많이 참조하는 신호
- [ ] middle-man.md — 미들 맨(81). 위임만 하는 클래스가 과도해지는 문제
- [ ] ch03-bad-smells-in-code.md — 챕터 3 종합 문서

---

## Chapter 4 — Building Tests

리팩토링의 안전망. Parallel Change 같은 기법이 성립하는 전제가 여기서 나온다.

- [ ] self-testing-code.md — 자가 검증 코드(85). 테스트가 사람 개입 없이 성공/실패를 스스로 판정해야 하는 이유
- [ ] test-first.md — 테스트를 먼저 쓰는 이유와 리팩토링 안전망으로서의 역할
- [ ] ch04-building-tests.md — 챕터 4 종합 문서

---

## Chapter 5 — Introducing the Catalog

카탈로그(6~12장)를 읽는 법에 대한 안내. 기법 하나 정도로 별도 파일을 만들 필요는 없고, 카탈로그 진입 전 개념만 종합 문서에 정리.

- [ ] ch05-introducing-the-catalog.md — 카탈로그 표기법(Mechanics, inverse of 등) 정리. 별도 키워드 파일 없이 이 문서 하나로 충분

---

## Chapter 6 — A First Set of Refactorings

가장 자주 쓰는 기본 기법들. 지호님이 처음 질문한 "메서드 복사 후 호출부 이전" 기법(Parallel Change 계열)의 토대가 되는 Extract/Move/Change Declaration이 여기 있다.

- [ ] extract-function.md — Extract Function(106). 코드 조각에 이름을 붙여 함수로 뽑아내는 가장 기본적인 기법
- [ ] inline-function.md — Inline Function(115). Extract Function의 역. 함수 호출보다 본문이 더 명확할 때 되돌리기
- [ ] extract-variable.md — Extract Variable(119). 복잡한 표현식에 이름을 붙여 변수로 뽑아내기
- [ ] inline-variable.md — Inline Variable(123). Extract Variable의 역
- [ ] change-function-declaration.md — Change Function Declaration(124). 함수 이름과 인자를 바꾸는 기법 — 지호님이 물어본 "메서드 시그니처를 안전하게 바꾸는 법"의 핵심
- [ ] parallel-change.md — Parallel Change / Migration Pattern. Change Function Declaration의 마이그레이션 절차(migration mechanics)로 책에 나오는, 지호님이 처음 질문한 바로 그 기법. 기존 메서드를 유지한 채 새 메서드를 만들고 호출부를 하나씩 옮긴 뒤 걷어내는 3단계
- [ ] encapsulate-variable.md — Encapsulate Variable(132). 데이터 접근에 함수를 끼워 넣어 통제권을 확보하는 기법
- [ ] rename-variable.md — Rename Variable(137). 변수 이름 변경 — Encapsulate Variable에 의존하는 경우가 많은 이유
- [ ] introduce-parameter-object.md — Introduce Parameter Object(140). 자주 함께 다니는 인자 뭉치를 객체 하나로 묶기
- [ ] combine-functions-into-class.md — Combine Functions into Class(144). 같은 데이터를 다루는 함수들을 클래스로 묶기
- [ ] combine-functions-into-transform.md — Combine Functions into Transform(149). 읽기 전용 데이터에 특화된 변형. 파생 데이터를 한 곳에서 계산해 붙이기
- [ ] split-phase.md — Split Phase(154). 하나의 로직을 순차적인 두 단계(예: 파싱과 계산)로 쪼개기
- [ ] ch06-a-first-set-of-refactorings.md — 챕터 6 종합 문서

---

## Chapter 7 — Encapsulation

데이터 구조를 캡슐화해서 모듈의 비밀을 숨기는 기법들. 지호님이 목표로 삼은 "domain 객체를 영역별로 나누는" 작업과 가장 밀접하다.

- [ ] encapsulate-record.md — Encapsulate Record(162). 레코드/구조체를 클래스로 감싸 접근을 통제하기
- [ ] encapsulate-collection.md — Encapsulate Collection(170). 컬렉션 필드를 그대로 노출하지 않고 캡슐화하기
- [ ] replace-primitive-with-object.md — Replace Primitive with Object(174). 단순 값이 자기만의 규칙을 갖기 시작하면 객체로 승격하기
- [ ] replace-temp-with-query.md — Replace Temp with Query(178). 임시 변수를 계산 함수(쿼리)로 바꿔 중복 계산 로직을 제거하기
- [ ] extract-class.md — Extract Class(182). 책임이 과해진 클래스를 둘로 쪼개기 — domain 객체를 영역별로 나눌 때 핵심이 되는 기법
- [ ] inline-class.md — Inline Class(186). Extract Class의 역
- [ ] hide-delegate.md — Hide Delegate(189). 위임 객체를 직접 노출하지 않고 감싸서 결합도를 낮추기
- [ ] remove-middle-man.md — Remove Middle Man(192). Hide Delegate의 역 — 위임 계층이 과해졌을 때 걷어내기
- [ ] substitute-algorithm.md — Substitute Algorithm(195). 알고리즘 전체를 더 명확한 것으로 통째로 교체하기
- [ ] ch07-encapsulation.md — 챕터 7 종합 문서

---

## Chapter 8 — Moving Features

프로그램 요소를 다른 컨텍스트로 옮기는 기법들. domain 객체 분리 작업에서 실제로 필드/함수를 새 클래스로 옮기는 실행 단계.

- [ ] move-function.md — Move Function(198). 함수를 더 적합한 클래스/모듈로 옮기기
- [ ] move-field.md — Move Field(207). 필드를 더 적합한 클래스로 옮기기
- [ ] move-statements-into-function.md — Move Statements into Function(213). 항상 같이 호출되는 코드를 함수 안으로 합치기
- [ ] move-statements-to-callers.md — Move Statements to Callers(217). Move Statements into Function의 역
- [ ] replace-inline-code-with-function-call.md — Replace Inline Code with Function Call(222). 이미 있는 함수와 같은 일을 하는 인라인 코드를 함수 호출로 교체하기
- [ ] slide-statements.md — Slide Statements(223). 관련 있는 코드끼리 붙도록 문장의 위치를 이동하기
- [ ] split-loop.md — Split Loop(227). 한 반복문이 여러 일을 하고 있으면 목적별로 쪼개기
- [ ] replace-loop-with-pipeline.md — Replace Loop with Pipeline(231). 반복문을 컬렉션 파이프라인 연산으로 교체하기
- [ ] remove-dead-code.md — Remove Dead Code(237). 더 이상 호출되지 않는 코드 제거
- [ ] ch08-moving-features.md — 챕터 8 종합 문서

---

## Chapter 9 — Organizing Data

데이터 구조 자체를 다루는 기법들.

- [ ] split-variable.md — Split Variable(240). 한 변수가 여러 역할로 재사용되면 역할별로 분리하기
- [ ] rename-field.md — Rename Field(244). 레코드 필드 이름 변경
- [ ] replace-derived-variable-with-query.md — Replace Derived Variable with Query(248). 다른 데이터로부터 계산 가능한 변수를 쿼리 함수로 대체하기
- [ ] change-reference-to-value.md — Change Reference to Value(252). 참조 객체를 값 객체로 바꾸기
- [ ] change-value-to-reference.md — Change Value to Reference(256). Change Reference to Value의 역
- [ ] ch09-organizing-data.md — 챕터 9 종합 문서

---

## Chapter 10 — Simplifying Conditional Logic

조건문을 다루기 쉽게 만드는 기법들.

- [ ] decompose-conditional.md — Decompose Conditional(260). 복잡한 조건과 분기 내용에 각각 이름을 붙여 함수로 뽑기
- [ ] consolidate-conditional-expression.md — Consolidate Conditional Expression(263). 같은 결과를 내는 여러 조건 검사를 하나로 통합하기
- [ ] replace-nested-conditional-with-guard-clauses.md — Replace Nested Conditional with Guard Clauses(266). 중첩 조건문을 조기 반환(guard clause)으로 평탄화하기
- [ ] replace-conditional-with-polymorphism.md — Replace Conditional with Polymorphism(272). 타입별 분기를 다형성으로 대체하기
- [ ] introduce-special-case.md — Introduce Special Case(289). null 체크 반복을 특수 케이스 객체(Null Object 등)로 대체하기
- [ ] introduce-assertion.md — Introduce Assertion(302). 코드가 암묵적으로 가정하는 조건을 명시적 단언으로 드러내기
- [ ] ch10-simplifying-conditional-logic.md — 챕터 10 종합 문서

---

## Chapter 11 — Refactoring APIs

모듈 간 경계(API)를 다루는 기법들.

- [ ] separate-query-from-modifier.md — Separate Query from Modifier(306). 값을 반환하면서 부수효과도 있는 함수를 조회 전용과 변경 전용으로 분리하기
- [ ] parameterize-function.md — Parameterize Function(310). 비슷한 로직을 가진 여러 함수를 매개변수 하나로 통합하기
- [ ] remove-flag-argument.md — Remove Flag Argument(314). 불리언 플래그로 동작을 분기하는 함수를 명시적인 함수 여러 개로 나누기
- [ ] preserve-whole-object.md — Preserve Whole Object(319). 한 객체에서 여러 값을 꺼내 넘기던 걸 객체 자체를 넘기는 방식으로 바꾸기
- [ ] replace-parameter-with-query.md — Replace Parameter with Query(324). 호출부에서 계산해 넘기던 인자를 함수 내부 쿼리로 대체하기
- [ ] replace-query-with-parameter.md — Replace Query with Parameter(327). Replace Parameter with Query의 역
- [ ] remove-setting-method.md — Remove Setting Method(331). 생성 이후 값이 바뀌지 않아야 하는 필드의 setter 제거
- [ ] replace-constructor-with-factory-function.md — Replace Constructor with Factory Function(334). 생성자보다 유연한 팩토리 함수로 객체 생성을 감싸기
- [ ] replace-function-with-command.md — Replace Function with Command(337). 함수를 커맨드 객체로 승격해 추가 제어(취소, undo 등)를 얻기
- [ ] replace-command-with-function.md — Replace Command with Function(344). Replace Function with Command의 역
- [ ] ch11-refactoring-apis.md — 챕터 11 종합 문서

---

## Chapter 12 — Dealing with Inheritance

상속 계층을 다루는 기법들. 마지막 챕터.

- [ ] pull-up-method.md — Pull Up Method(350). 여러 서브클래스에 중복된 메서드를 상위 클래스로 올리기
- [ ] pull-up-field.md — Pull Up Field(353). 여러 서브클래스에 중복된 필드를 상위 클래스로 올리기
- [ ] pull-up-constructor-body.md — Pull Up Constructor Body(355). 서브클래스 생성자의 공통 부분을 상위 클래스 생성자로 올리기
- [ ] push-down-method.md — Push Down Method(359). Pull Up Method의 역 — 일부 서브클래스에만 필요한 메서드를 아래로 내리기
- [ ] push-down-field.md — Push Down Field(361). Pull Up Field의 역
- [ ] replace-type-code-with-subclasses.md — Replace Type Code with Subclasses(362). 타입을 나타내는 필드를 서브클래스 구조로 대체하기
- [ ] remove-subclass.md — Remove Subclass(369). Replace Type Code with Subclasses의 역 — 서브클래스가 별 차이를 안 만들 때 제거
- [ ] extract-superclass.md — Extract Superclass(375). 공통점이 있는 클래스들에서 상위 클래스를 뽑아내기
- [ ] collapse-hierarchy.md — Collapse Hierarchy(380). 상위/하위 클래스가 더 이상 다르지 않을 때 하나로 합치기
- [ ] replace-subclass-with-delegate.md — Replace Subclass with Delegate(381). 상속을 위임으로 대체하기 — 상속의 제약(단일 상속, 컴파일 타임 고정)을 벗어나는 기법
- [ ] replace-superclass-with-delegate.md — Replace Superclass with Delegate(399). 상위/하위 관계 자체를 위임 관계로 대체하기
- [ ] ch12-dealing-with-inheritance.md — 챕터 12 종합 문서

---

진행 방식: Chapter 1 → 2 → 3 → 4 → 5까지 원칙을 먼저 잡은 뒤, 6 → 7 → 8 → 9 → 10 → 11 → 12 카탈로그 순서로 진행 권장.
다만 카탈로그 파트(6~12장)는 참고서 성격이 강하므로, 지호님이 실무에서 마주친 기법이 있으면 순서를 건너뛰고 먼저 다뤄도 무방하다.
완료된 항목은 [x]로 표시. 로드맵에는 완료 후 실제 생성된 파일만 남긴다.
