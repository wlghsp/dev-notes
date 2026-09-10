# Week 4 준비: 근거형 질문 직결 개념

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
미션: 결제(Payment) 도메인을 SINGLE_TABLE과 JOINED 두 상속 전략으로 각각 구현하고, 같은 H2 데이터·조회 조건에서 스키마·다형성 조회 SQL·변경 비용을 비교해 ADR로 남긴다.

정답 없음. Week 1~3에서 쓴 `JpaObservation`(SQL 관측 도구)을 재사용해서 "무엇을 관찰할지"만 짚는다. Payment 도메인은 이번 주차에 새로 만드는 것이라 아직 코드가 없다 — 이번 prep-questions는 설계 전에 답해야 하는 개념 질문 위주다.

---

## A-1. 근거형 질문 직결 개념

정답은 안 적는다. 각 질문에서 무엇을 코드로 직접 확인해야 답이 나오는지만 짚는다.

- 상속 매핑 전략을 선택할 때 조회 성능과 스키마 정규화를 어떻게 비교했는가?
  - 확인할 것: SINGLE_TABLE(한 테이블에 모든 서브타입 컬럼을 모음)과 JOINED(부모/자식 테이블을 조인)가 각각 만드는 실제 테이블 스키마를 H2에서 직접 확인하고, 같은 조회를 실행했을 때 SQL이 몇 개의 테이블을 건드리는지(B-1)

:

- SINGLE_TABLE의 nullable 컬럼 비용과 JOINED의 조인 비용을 이번 모델에서 어떻게 확인하는가?
  - 확인할 것: SINGLE_TABLE에서는 서브타입별로만 쓰는 컬럼이 다른 서브타입 행에서는 전부 NULL로 남는다는 것을 실제 테이블 데이터로 확인, JOINED에서는 서브타입 조회 시 부모+자식 테이블 JOIN이 몇 번 발생하는지 SQL 로그로 확인(B-2)

:

- 다형성 조회 SQL이 두 전략에서 어떻게 다르고, 이 H2 실험 결과를 어디까지 일반화할 수 있는가?
  - 확인할 것: 같은 "모든 결제 조회" 요청을 두 전략 각각에서 실행했을 때 실제 SQL 형태(SINGLE_TABLE은 단일 SELECT + WHERE dtype, JOINED는 LEFT OUTER JOIN 체인)가 어떻게 다른지, 그리고 H2와 실제 운영 DB(MySQL 등)에서 옵티마이저 동작이 달라질 수 있다는 한계를 어떻게 명시할지(B-3)

:

- 두 매핑안 중 버린 안이 더 적합해지는 요구사항은 무엇인가?
  - 확인할 것: 서브타입 고유 컬럼이 많고 자주 추가되는지(JOINED가 유리), 서브타입 간 컬럼 겹침이 적고 조회 성능이 중요한지(SINGLE_TABLE이 유리) — 이번 결제 도메인의 실제 특성(카드/계좌이체/포인트 등 서브타입이 얼마나 다른 필드를 갖는지)에 비춰 판단(B-4)

:

이 4가지는 근거형 질문 1~4와 대응한다.

## A-2. 실험 설계 전에 확정해야 하는 것

- Payment 도메인을 최소 재현 모델로 어떻게 설계할 것인가? (부모 클래스 Payment + 서브타입 2개 이상, 예: CardPayment/BankTransferPayment)

:

- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`과 `@Inheritance(strategy = InheritanceType.JOINED)`를 각각 어느 클래스에 붙이는가? SINGLE_TABLE에서 서브타입 구분 컬럼(`@DiscriminatorColumn`)은 왜 필요한가?

:

- "같은 H2 데이터와 조회 조건"이라는 말은 두 전략을 비교할 때 구체적으로 무엇을 고정해야 하는가? (같은 개수의 서브타입별 row, 같은 조회 쿼리, `JpaObservation.reset()` 호출 시점)

:

## A-3. 실험 중 부딪힐 것들

- SINGLE_TABLE에서 서브타입 고유 컬럼에 `not null` 제약을 걸면 무슨 문제가 생기는가? (다른 서브타입 row에서는 그 컬럼이 항상 NULL이어야 하므로)

:

- JOINED에서 다형성 조회(부모 타입으로 전체 조회) 시 발생하는 SQL이 서브타입 N개일 때 N번 조인되는가, 아니면 다른 전략(UNION 등)을 쓰는가? Hibernate가 실제로 어떤 SQL을 만드는지 로그로 확인해야 하는 지점

:

- INSERT 비용 비교: SINGLE_TABLE은 1개 테이블에 1번 INSERT, JOINED는 부모/자식 테이블에 각각 INSERT가 필요하다는 것을 실제 SQL 로그(`JpaObservation`)로 어떻게 확인하는가?

:

- 변경 비용(스키마 변경)을 비교하려면 무엇을 시나리오로 삼아야 하는가? (예: 서브타입에 컬럼 하나를 추가했을 때 SINGLE_TABLE은 기존 테이블에 컬럼 추가, JOINED는 자식 테이블에만 컬럼 추가 — 영향 범위가 다름)

:

## A-4. 선택 과제(값 타입 불변성 또는 복합 키)를 고를지 판단

- 값 타입 불변성 실험은 무엇을 검증하는 것인가? (`@Embeddable` 값 타입을 공유 참조로 두었을 때 의도치 않은 변경이 다른 엔티티에도 반영되는 문제)

:

- 복합 키 계약 실험은 무엇을 검증하는 것인가? (`@EmbeddedId` 또는 `@IdClass`를 쓸 때 `equals`/`hashCode` 구현이 왜 필수인지)

:

- 이번 주차는 필수 항목(SINGLE_TABLE vs JOINED 비교 + ADR)만으로도 분량이 크다. 선택 과제를 할지는 필수를 먼저 끝낸 뒤 판단한다

:

## A-5. ADR 작성 전 확인

- ADR(Architecture Decision Record)에 반드시 들어가야 하는 항목은 무엇인가? (배경/문제, 고려한 옵션들, 선택한 옵션과 이유, 트레이드오프, 버려진 옵션이 다시 적합해지는 조건)

:

- "강의 정답을 근거 없이 복사한 흔적이 없어야 한다"는 통과 기준을 지키려면, ADR의 각 판단에 무엇을 근거로 붙여야 하는가? (이번 실험에서 직접 관찰한 스키마/SQL/측정값)

:

## A-6. 리뷰 반영 체크리스트 (1~3주차 review-notes.md 누적)

- week1에서 지적받고 week3에서 다시 지적받은 "## 리뷰 반영" 공백 문제를 이번 주차에 어떻게 방지할 것인가?

:

- self-check가 실패하면 `gh run view <run-id> --job <job-id> --log`로 원인을 직접 확인하고, "채점 Workflow/challenge.json 변경 금지" 에러라면 `git merge origin/main`으로 최신화 후 재push한다 — week3에서 실제로 겪은 절차를 이번 주차 시작 전에 먼저 적용할 것

:

- PR의 "Files changed"에서 새로 추가한 테스트 파일들이 전부 정상적으로 diff에 잡히는지 확인 (week3에서 일부 누락되어 지적받음)

:
