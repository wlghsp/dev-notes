# week-04 evidence 드래프트

다 채운 뒤 challenge-jpa-deep-dive-2026-08-wlghsp-r8/evidence/week-04__weekly-pr.md 로 옮긴다.

## 변경

1. 같은 결제 도메인(`Payment` 부모, `CardPayment`/`BankTransferPayment` 자식)을 SINGLE_TABLE과 JOINED 두 전략으로 각각 최소 재현했다.
  - `PaymentSingleTable`(`@Inheritance(SINGLE_TABLE)`, `@DiscriminatorColumn(payment_type)`) + `CardPaymentSingleTable`, `BankTransferPaymentSingleTable`
  - `PaymentJoined`(`@Inheritance(JOINED)`) + `CardPaymentJoined`, `BankTransferPaymentJoined`
  - `@Inheritance`는 계층 전체에 한 번만 적용되는 설정이라 같은 클래스로 두 전략을 동시에 비교할 수 없어서, 전략별로 클래스와 테이블을 완전히 분리했다.
2. week4 패키지에 테스트 2개 클래스, 총 5개 테스트 작성:
  - `PaymentSingleTableTest` (3개) — INSERT 1회, 타 서브타입 컬럼 NULL, 다형성 조회 시 조인 없음
  - `PaymentJoinedTest` (2개) — INSERT 부모/자식 테이블에 각각 발생, 다형성 조회 시 LEFT OUTER JOIN 발생

## 검증

### 실행 명령

```
mvn test -Dtest=PaymentSingleTableTest,PaymentJoinedTest
```

### 결과 요약

직접 실행하고 아래를 채우세요.

- 테스트 통과 개수
- 생성된 스키마(DDL)를 SINGLE_TABLE과 JOINED 나란히 캡처해서 비교 — 힌트: `payment_type` 같은 discriminator 컬럼이 어디에 있는지, nullable 컬럼이 어디에 몰리는지 보세요.
- INSERT 로그 비교 — SINGLE_TABLE은 몇 번, JOINED는 몇 번, 어느 테이블에 찍히는지.
- 다형성 조회(`select p from PaymentXxx p`) SQL을 두 전략 각각 캡처 — JOIN이 몇 개 붙는지, discriminator/PK로 타입을 어떻게 구분하는지 SQL을 눈으로 직접 확인하세요.

## 선택 근거

- 두 전략을 완전히 별도 엔티티/테이블로 분리해서 구현했다. `@Inheritance`는 상속 계층 전체에 한 번만 적용되는 설정이라, 같은 `Payment` 클래스로는 SINGLE_TABLE과 JOINED를 동시에 보여줄 수 없다. 그래서 같은 프로젝트 안에서 두 전략을 나란히 비교하려면 클래스 자체를 나누는 것이 유일한 방법이었다.
- NOT NULL 제약을 SINGLE_TABLE 서브타입 컬럼에 직접 걸어보지는 않았다. 대신 생성된 DDL에서 `card_number`, `account_number` 등 서브타입 전용 컬럼이 전부 nullable로 잡힌다는 사실만으로, 제약을 걸면 다른 서브타입 row에서 위반이 날 수밖에 없다는 결론이 구조적으로 따라온다.
  - [증빙: SINGLE_TABLE DDL과 JOINED DDL을 나란히 붙여넣기]
- H2에서 확인한 것은 SQL의 구조(조인 유무, 조인 개수, discriminator 사용 여부)이지 절대적인 실행 비용(ms)이 아니다. 구조는 SQL 표준에 가까운 동작이라 다른 DB 엔진에도 일반화할 수 있다고 보지만, 그 구조 차이가 실제로 몇 ms의 차이를 만드는지는 옵티마이저와 인덱스 동작이 다른 MySQL 등에서 별도로 실측해야 한다 — 이번 실험은 구조 확인에서 멈췄다는 한계를 남긴다.

## ADR

### 배경

같은 결제 도메인을 SINGLE_TABLE과 JOINED 두 상속 전략으로 각각 구현했다. 실제로 이 프로젝트에 적용한다면 어느 쪽을 선택할지 결정해야 한다.

### 결정

이번 최소 재현 모델의 결제 도메인(서브타입 2개, 서브타입 전용 컬럼 각 2개 내외)에서는 **JOINED**를 선택한다.

### 근거

- 결제는 서브타입별 정합성이 중요한 도메인이다. `card_number`, `account_number` 같은 서브타입 전용 컬럼에 NOT NULL 제약을 DB 레벨에서 강제할 수 있어야 하는데, 이건 JOINED에서만 가능하다. SINGLE_TABLE은 이 검증 책임을 전부 애플리케이션 코드가 떠안아야 한다.
- 지금은 서브타입이 2개뿐이라 JOINED의 조인 비용이 아직 크지 않다. 조인 개수는 서브타입 수에 비례해 늘어나므로, 지금 규모에서는 정규화(무결성)의 이득이 조인 비용보다 크다고 판단했다.

### 버린 안: SINGLE_TABLE

- 조회가 압도적으로 빈번하고 조인 비용을 감내하기 어려운 상황, 또는 서브타입 전용 컬럼의 무결성을 애플리케이션 레벨 검증만으로 충분히 감당할 수 있는 경우라면 SINGLE_TABLE이 더 적합해진다.
- 서브타입 수가 늘어나 JOINED의 조인 개수가 부담스러워지고, 서브타입 전용 컬럼이 소수라 nullable 컬럼 낭비가 크지 않다면 SINGLE_TABLE로 재평가할 수 있다.

### 재평가 조건

- 서브타입이 5개 이상으로 늘어나 JOINED 다형성 조회의 조인 개수가 부담될 때
- 운영 DB(MySQL 등)에서 실측했을 때 JOINED의 조인 비용이 무결성 이득을 상회한다고 확인될 때

## 근거형 질문

미션 가이드의 질문마다 최초 판단 → 연결한 코드·테스트·로그 → 검증 후 답변 순서로 적습니다.

1. 상속 매핑 전략을 선택할 때 조회 성능과 스키마 정규화를 어떻게 비교했나요?
- 판단: 조회 성능(조인 유무)과 정규화(nullable 컬럼, NOT NULL 제약 가능 여부)는 트레이드오프 관계이며, 어느 쪽도 절대적으로 우월하지 않다.
- 근거: [증빙: SINGLE_TABLE/JOINED DDL 비교, `polymorphic_query_has_no_join` / `polymorphic_query_has_join` 테스트 결과와 실제 SQL]
- 답변: 이번 도메인은 서브타입이 적고(2개) 정합성이 중요해 정규화 쪽에 무게를 둬 JOINED를 선택했다. 서브타입이 많아지고 조회가 압도적으로 빈번해지면 판단이 바뀔 수 있다.

2. SINGLE_TABLE의 nullable 컬럼 비용과 JOINED의 조인 비용을 이번 모델에서 어떻게 확인했나요?
- 판단: nullable 컬럼 비용은 DDL(생성된 컬럼 목록)로, 조인 비용은 발생한 SQL의 JOIN 개수로 직접 셀 수 있다.
- 근거: [증빙: SINGLE_TABLE DDL의 nullable 컬럼 목록, JOINED 다형성 조회 SQL의 JOIN 개수]
- 답변: 두 비용 모두 정성적 방향(있다/없다)은 로그로 명확히 확인했다. 다만 이 nullable 컬럼과 조인이 실제 운영에서 몇 ms의 비용 차이를 만드는지는 H2 소규모 데이터로는 측정하지 않았다 — 구조적 차이만 확인했고 절대 비용은 확인 불가로 남긴다.

3. 다형성 조회 SQL이 두 전략에서 어떻게 달랐고 H2 실험 결과를 어디까지 일반화할 수 있나요?
- 판단: SINGLE_TABLE은 단일 SELECT + discriminator column 판별, JOINED는 부모 테이블에 서브타입 수만큼 LEFT OUTER JOIN을 붙이고 각 서브타입 PK의 not-null 여부로 실제 타입을 판별할 것이다.
- 근거: [증빙: 두 전략의 다형성 조회 SQL 로그를 나란히 붙여넣기]
- 답변: SQL 구조(조인 유무, 조인 개수, discriminator 사용 여부)는 SQL 표준에 가까운 동작이라 다른 DB 엔진에도 일반화할 수 있다고 본다. 하지만 이 구조 차이가 실제로 몇 ms의 응답 시간 차이를 만드는지는 H2 옵티마이저와 MySQL 옵티마이저가 달라 일반화할 수 없다 — 이번 실험은 구조 확인에서 멈췄다.

4. 두 매핑안 중 버린 안이 더 적합해지는 요구사항은 무엇인가요?
- 판단: 버린 안(SINGLE_TABLE)은 조회 빈도가 매우 높고 서브타입 전용 컬럼의 무결성을 애플리케이션 레벨에서 충분히 감당할 수 있을 때 더 적합해진다.
- 근거: 미션 가이드의 선택 기준과 이번 도메인 구조(서브타입 2개, 전용 컬럼 소수)를 비교.
- 답변: 서브타입이 지금보다 훨씬 많아지거나(예: 5개 이상), 다형성 조회가 트래픽 대부분을 차지하고 단일 서브타입 단독 조회가 드물어지면, JOINED의 조인 비용이 정규화 이득보다 커질 수 있어 SINGLE_TABLE로 재평가할 조건이 된다.

## 리뷰 반영

자동 리뷰 수신 전
