# Week 4 준비: 근거형 질문 직결 개념

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
미션: 결제(Payment) 도메인을 SINGLE_TABLE과 JOINED 두 상속 전략으로 각각 구현하고, 같은 H2 데이터·조회 조건에서 스키마·다형성 조회 SQL·변경 비용을 비교해 ADR로 남긴다.

정답 없음. Week 1~3에서 쓴 `JpaObservation`(SQL 관측 도구)을 재사용해서 "무엇을 관찰할지"만 짚는다. Payment 도메인은 이번 주차에 새로 만드는 것이라 아직 코드가 없다 — 이번 prep-questions는 설계 전에 답해야 하는 개념 질문 위주다.

---

## A-1. 근거형 질문 직결 개념

정답은 안 적는다. 각 질문에서 무엇을 코드로 직접 확인해야 답이 나오는지만 짚는다.

- 상속 매핑 전략을 선택할 때 조회 성능과 스키마 정규화를 어떻게 비교했는가?
  - 확인할 것: SINGLE_TABLE(한 테이블에 모든 서브타입 컬럼을 모음)과 JOINED(부모/자식 테이블을 조인)가 각각 만드는 실제 테이블 스키마를 H2에서 직접 확인하고, 같은 조회를 실행했을 때 SQL이 몇 개의 테이블을 건드리는지(B-1)

: SINGLE_TABLE: 조인 없음 → 조회는 항상 빠름. 대신 서브클래스 전용 컬럼이 전부 nullable, 타입 discriminator 컬럼 필요 → 정규화 위반
JOINED: 정규화는 지켜지지만 조회할 때마다 부모-자식 조인 필요 → 상속 깊이가 깊어지거나 조회 빈도가 높으면 성능 손해가 구조적으로 예정되어 있음
TABLE_PER_CLASS: 다형 쿼리 시 UNION 필요 → 이것도 구조적으로 불리함이 예측 가능

방향성(어느 쪽이 유리한가)은 구조만 봐도 논리적으로 도출 가능하지만, 실제 차이의 크기는 EXPLAIN이나 벤치마크로 확인이 필요하다.


- SINGLE_TABLE의 nullable 컬럼 비용과 JOINED의 조인 비용을 이번 모델에서 어떻게 확인하는가?
  - 확인할 것: SINGLE_TABLE에서는 서브타입별로만 쓰는 컬럼이 다른 서브타입 행에서는 전부 NULL로 남는다는 것을 실제 테이블 데이터로 확인, JOINED에서는 서브타입 조회 시 부모+자식 테이블 JOIN이 몇 번 발생하는지 SQL 로그로 확인(B-2)

SINGLE_TABLE의 nullable 컬럼 비용 확인
- 테이블 생성 후 DESCRIBE 또는 스키마 조회로 서브클래스별 컬럼들이 전부 NULL 허용으로 잡히는지 직접 확인
- 실제 데이터 넣고 특정 서브타입 row를 봤을 때 다른 서브타입 컬럼들이 NULL로 채워지는 걸 눈으로 확인
- 이건 성능 비용이라기보단 스키마 품질/저장공간 비용이라, information_schema나 테이블 크기 비교 정도면 충분히 실증 가능

JOINED의 조인 비용 확인
- 이건 진짜 실측이 필요한 영역 — 부모/자식 각각에 데이터를 채운 뒤 다형 쿼리(부모 타입으로 조회)를 날렸을 때 EXPLAIN으로 실행 계획 확인

- 다형성 조회 SQL이 두 전략에서 어떻게 다르고, 이 H2 실험 결과를 어디까지 일반화할 수 있는가?
  - 확인할 것: 같은 "모든 결제 조회" 요청을 두 전략 각각에서 실행했을 때 실제 SQL 형태(SINGLE_TABLE은 단일 SELECT + WHERE dtype, JOINED는 LEFT OUTER JOIN 체인)가 어떻게 다른지, 그리고 H2와 실제 운영 DB(MySQL 등)에서 옵티마이저 동작이 달라질 수 있다는 한계를 어떻게 명시할지(B-3)

SQL 차이: JOINED는 부모+자식 테이블을 조인해서 조회하고(자식 많을수록 조인도 늘어남), SINGLE_TABLE은 한 테이블에서 discriminator 컬럼으로만 구분해 조인이 없음

일반화 범위: 조인 구조 자체(형태)는 어떤 DB에서든 똑같이 적용되지만, H2에서 측정한 실행 시간/성능 차이는 실제 운영 DB(MySQL 등)로 일반화 안 됨 — 옵티마이저와 인덱스 동작이 다르기 때문

- 두 매핑안 중 버린 안이 더 적합해지는 요구사항은 무엇인가?
  - 확인할 것: 서브타입 고유 컬럼이 많고 자주 추가되는지(JOINED가 유리), 서브타입 간 컬럼 겹침이 적고 조회 성능이 중요한지(SINGLE_TABLE이 유리) — 이번 결제 도메인의 실제 특성(카드/계좌이체/포인트 등 서브타입이 얼마나 다른 필드를 갖는지)에 비춰 판단(B-4)

SINGLE_TABLE이 더 적합한 경우
- 조회 성능이 최우선이고 조인 비용을 감내하기 어려울 때(트래픽 많고 상속 깊이 얕음)
- 서브클래스 전용 컬럼 수가 적어서 nullable 컬럼 낭비가 크지 않을 때
- 다형성 쿼리가 매우 빈번한데 서브타입별 데이터 삽입/수정은 상대적으로 드물 때

JOINED가 더 적합한 경우
- 서브클래스마다 컬럼이 많고 서로 겹치지 않아서 nullable 컬럼이 과도하게 늘어날 때
- 서브타입 하나만 단독으로 자주 조회/수정하는 패턴이 많을 때
- 정규화·데이터 무결성이 중요해서 NOT NULL 제약을 서브클래스별로 강제하고 싶을 때

이 4가지는 근거형 질문 1~4와 대응한다.

## A-2. 실험 설계 전에 확정해야 하는 것

- Payment 도메인을 최소 재현 모델로 어떻게 설계할 것인가? (부모 클래스 Payment + 서브타입 2개 이상, 예: CardPayment/BankTransferPayment)

:
부모 클래스 Payment
- 공통 필드: id, amount, status, createdAt
- `@Inheritance(strategy = ...)`를 여기 선언

서브타입 (최소 2개, 컬럼이 서로 겹치지 않아야 nullable 비용이 드러남)
- CardPayment: cardNumber, installmentMonths
- BankTransferPayment: bankCode, accountNumber

- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`과 `@Inheritance(strategy = InheritanceType.JOINED)`를 각각 어느 클래스에 붙이는가? SINGLE_TABLE에서 서브타입 구분 컬럼(`@DiscriminatorColumn`)은 왜 필요한가?

`@Inheritance`는 항상 부모(최상위) 클래스에만 붙임. 서브클래스에는 붙이지 않음.

SINGLE_TABLE에서 `@DiscriminatorColumn`이 필요한 이유: 부모+모든 자식이 테이블 하나에 몰리기 때문에, row 하나만 봐서는 어떤 서브타입인지 구분할 방법이 없음. NULL 여부로 추론하는 건 취약하고 JPA 스펙상 신뢰할 수 없음. 그래서 discriminator 컬럼(기본 DTYPE)에 타입 이름을 저장해서 Hibernate가 이 값을 보고 매핑할 자바 클래스를 결정함.

JOINED는 필요 없음 — 자식 테이블에 row가 있다는 사실 자체가 타입 정보를 담고 있기 때문.

- "같은 H2 데이터와 조회 조건"이라는 말은 두 전략을 비교할 때 구체적으로 무엇을 고정해야 하는가? (같은 개수의 서브타입별 row, 같은 조회 쿼리, `JpaObservation.reset()` 호출 시점)

1. 서브타입별 row 개수 — 양쪽 전략에 동일하게
2. 조회 쿼리 — 같은 다형성 조회(부모 타입으로 findAll), 같은 조건/정렬/페이징
3. `JpaObservation.reset()` 호출 시점 — 데이터 삽입·flush 이후, 측정 대상 조회 직전으로 양쪽 동일하게 맞춤

## A-3. 실험 중 부딪힐 것들

- SINGLE_TABLE에서 서브타입 고유 컬럼에 `not null` 제약을 걸면 무슨 문제가 생기는가? (다른 서브타입 row에서는 그 컬럼이 항상 NULL이어야 하므로)

`cardNumber not null`을 걸면 같은 테이블의 BankTransferPayment row는 cardNumber가 없는 게 정상인데도 
제약 위반으로 INSERT가 실패함. "이 서브타입일 때만 필수"를 테이블 레벨에서 표현할 방법이 없어서, 서브타입 고유 컬럼은
구조적으로 nullable일 수밖에 없음. 무결성 검증은 애플리케이션이 떠안아야 함.

- JOINED에서 다형성 조회(부모 타입으로 전체 조회) 시 발생하는 SQL이 서브타입 N개일 때 N번 조인되는가, 아니면 다른 전략(UNION 등)을 쓰는가? Hibernate가 실제로 어떤 SQL을 만드는지 로그로 확인해야 하는 지점

Hibernate는 부모 테이블 기준으로 모든 서브타입 테이블을 LEFT OUTER JOIN으로 붙임(UNION 아님). 서브타입 N개면 조인도 N개. `show-sql`로 다형성 조회 실행 시 SQL에 LEFT OUTER JOIN이 몇 번 나오는지 직접 세어서 확인 

- INSERT 비용 비교: SINGLE_TABLE은 1개 테이블에 1번 INSERT, JOINED는 부모/자식 테이블에 각각 INSERT가 필요하다는 것을 실제 SQL 로그(`JpaObservation`)로 어떻게 확인하는가?

CardPayment 하나를 persist + flush 한 직후 `observation.statements()` 확인. SINGLE_TABLE은 INSERT 1건, JOINED는 부모 테이블 INSERT 1건 + 자식 테이블 INSERT 1건 총 2건. 이 개수 차이를 assertion으로 검증

- 변경 비용(스키마 변경)을 비교하려면 무엇을 시나리오로 삼아야 하는가? (예: 서브타입에 컬럼 하나를 추가했을 때 SINGLE_TABLE은 기존 테이블에 컬럼 추가, JOINED는 자식 테이블에만 컬럼 추가 — 영향 범위가 다름)

CardPayment에 필드 하나(예: cardIssuer)를 추가했을 때 생성되는 DDL을 비교. SINGLE_TABLE은 기존 단일 테이블에 컬럼 추가 → 다른 서브타입 row에도 영향(NULL로 채워짐). JOINED는 card_payment 자식 테이블에만 컬럼 추가 → BankTransferPayment와 무관, 영향 범위가 좁음. `ddl-auto=create`로 양쪽 DDL을 뽑아 비교

## A-4. 선택 과제(값 타입 불변성 또는 복합 키)를 고를지 판단

- 값 타입 불변성 실험은 무엇을 검증하는 것인가? (`@Embeddable` 값 타입을 공유 참조로 두었을 때 의도치 않은 변경이 다른 엔티티에도 반영되는 문제)

값 타입은 식별자가 없고 소유 엔티티에 종속되는데, 같은 값 타입 인스턴스를 두 엔티티가 참조로 공유하면 한쪽에서 값을 바꿨을 때 다른 엔티티의 값도 같이 바뀜(같은 객체를 참조하므로). 값 타입을 불변으로 만들고(setter 없이 생성자로만 값 설정) 공유하지 않아야 한다는 걸 검증.

- 복합 키 계약 실험은 무엇을 검증하는 것인가? (`@EmbeddedId` 또는 `@IdClass`를 쓸 때 `equals`/`hashCode` 구현이 왜 필수인지)

JPA는 엔티티를 식별자로 구분하는데, 복합 키는 여러 필드를 묶은 객체라 Java 기본 equals/hashCode(참조 비교)로는
"같은 키인데 다른 객체"를 다른 것으로 취급함. 1차 캐시 조회, 컬렉션 중복 판단이 깨짐. 복합 키 클래스는 필드 값 기반 equals/hashCode를 직접 구현해야 한다는 걸 검증.

- 이번 주차는 필수 항목(SINGLE_TABLE vs JOINED 비교 + ADR)만으로도 분량이 크다. 선택 과제를 할지는 필수를 먼저 끝낸 뒤 판단한다

: 필수 항목(비교 실험 + ADR)을 먼저 끝내고, 남는 시간에만 선택 과제 착수

## A-5. ADR 작성 전 확인

- ADR(Architecture Decision Record)에 반드시 들어가야 하는 항목은 무엇인가? (배경/문제, 고려한 옵션들, 선택한 옵션과 이유, 트레이드오프, 버려진 옵션이 다시 적합해지는 조건)

배경/문제, 고려한 옵션들(SINGLE_TABLE, JOINED, 필요하면 TABLE_PER_CLASS), 선택한 옵션과 이유, 트레이드오프, 버려진 옵션이 다시 적합해지는 조건. 다섯 항목 모두 필수

- "강의 정답을 근거 없이 복사한 흔적이 없어야 한다"는 통과 기준을 지키려면, ADR의 각 판단에 무엇을 근거로 붙여야 하는가? (이번 실험에서 직접 관찰한 스키마/SQL/측정값)

: 일반론으로 적지 말고, 이번 실험에서 직접 뽑은 스키마 DDL, JpaObservation으로 관찰한 실제 SQL 로그, 쿼리 개수 같은 구체적 수치를 각 판단 옆에 근거로 붙임

## A-6. 리뷰 반영 체크리스트 (1~3주차 review-notes.md 누적)

- week1에서 지적받고 week3에서 다시 지적받은 "## 리뷰 반영" 공백 문제를 이번 주차에 어떻게 방지할 것인가?

"자동 리뷰 수신 전"만 적어두고 끝내지 말고, 리뷰 코멘트를 받으면 즉시 무엇을 어떻게 반영했는지 + 재검증 결과를 채워 넣는 걸 커밋 전 체크리스트에 명시.


- self-check가 실패하면 `gh run view <run-id> --job <job-id> --log`로 원인을 직접 확인하고, "채점 Workflow/challenge.json 변경 금지" 에러라면 `git merge origin/main`으로 최신화 후 재push한다 — week3에서 실제로 겪은 절차를 이번 주차 시작 전에 먼저 적용할 것

작업 시작 전 `git merge origin/main`부터 먼저 실행. self-check 실패 시 바로 재시도하지 말고 `gh run view <run-id> --job <job-id> --log`로 로그부터 확인.

- PR의 "Files changed"에서 새로 추가한 테스트 파일들이 전부 정상적으로 diff에 잡히는지 확인 (week3에서 일부 누락되어 지적받음)

PR 올리기 전 `git status`와 `gh pr diff`로 새로 만든 테스트 파일이 실제로 diff에 포함됐는지 확인(untracked 상태로 add 안 된 채 남아있지 않은지).

