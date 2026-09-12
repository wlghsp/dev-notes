# week-04 evidence 드래프트

다 채운 뒤 challenge-jpa-deep-dive-2026-08-wlghsp-r8/evidence/week-04__weekly-pr.md 로 옮긴다.

## 변경

1. 결제 도메인(`Payment` 부모, `CardPayment`/`BankTransferPayment` 자식)을 SINGLE_TABLE, JOINED 두 전략으로 각각 최소 재현했다.
  - `PaymentSingleTable`(`@Inheritance(SINGLE_TABLE)`, `@DiscriminatorColumn(payment_type)`) + `CardPaymentSingleTable`, `BankTransferPaymentSingleTable`
  - `PaymentJoined`(`@Inheritance(JOINED)`) + `CardPaymentJoined`, `BankTransferPaymentJoined`
  - `@Inheritance`는 계층당 하나만 적용 가능해 같은 클래스로 두 전략을 동시에 비교할 수 없으므로 클래스·테이블을 완전히 분리했다.
2. week4 패키지에 테스트 2개 클래스, 총 5개 작성:
  - `PaymentSingleTableTest` (3개) — INSERT 1회, 타 서브타입 컬럼 NULL, 다형성 조회 시 조인 없음
  - `PaymentJoinedTest` (2개) — INSERT 부모/자식 테이블에 각각 발생, 다형성 조회 시 LEFT OUTER JOIN 발생

## 검증

### 실행 명령

```
mvn test -Dtest=PaymentSingleTableTest,PaymentJoinedTest
```

### 결과 요약

- 2개 클래스 총 5개 테스트 모두 통과 (`PaymentSingleTableTest` 3개, `PaymentJoinedTest` 2개).

### SINGLE_TABLE vs JOINED DDL·INSERT·다형성 조회 로그

미션 필수 제출 증거(DDL 비교, INSERT 로그 비교, 다형성 조회 SQL 비교)에 해당하는 산출물이다. 원본 SQL 로그는 evidence/week-04__weekly-pr-sql-logs.md 파일에 별도로 남겼다. `mvn test -Dtest=PaymentSingleTableTest,PaymentJoinedTest`로 재현 가능하다.

요약:
- DDL — SINGLE_TABLE은 테이블 1개에 discriminator(`payment_type`, not null)와 서브타입 컬럼이 다 모여있고 nullable. JOINED는 discriminator 없이 서브타입 컬럼이 자식 테이블로 분리되어 not null 가능. 자식 PK는 부모 PK 참조 FK.
- INSERT — SINGLE_TABLE은 row당 1회, 단일 테이블(3건 → 3회). JOINED는 row당 2회, 부모+자식 각 1회(카드 2건+계좌이체 1건 → payment_joined 3회, card_payment_joined 2회, bank_transfer_payment_joined 1회).
- 다형성 조회 — SINGLE_TABLE은 JOIN 없음. JOINED는 서브타입 수(2개)만큼 LEFT OUTER JOIN, 자식 PK의 null 여부를 CASE WHEN으로 판별.

## 선택 근거

- NOT NULL 제약을 SINGLE_TABLE 서브타입 컬럼에 직접 걸어보진 않았지만, DDL에서 `card_number`, `account_number` 등이 전부 nullable로 잡힌다는 사실만으로 제약 시 다른 서브타입 row에서 위반이 날 수밖에 없다는 결론이 구조적으로 따라온다.
- H2에서 확인한 건 SQL 구조(조인 유무·개수, discriminator 사용 여부)이지 절대 실행 비용(ms)이 아니다. 구조는 다른 DB 엔진에도 일반화 가능하다고 보지만, 실제 ms 차이는 옵티마이저·인덱스가 다른 MySQL 등에서 별도 실측이 필요하다 — 이번 실험은 구조 확인에서 멈춘 한계로 남긴다.

## ADR

### 배경

결제 도메인을 SINGLE_TABLE·JOINED 두 전략으로 구현했다. 실제 적용 시 어느 쪽을 선택할지 결정해야 한다.

### 결정

이번 규모(서브타입 2개, 전용 컬럼 각 2개 내외)에서는 **JOINED**를 선택한다.

### 근거

- 결제는 서브타입별 정합성이 중요하다. `card_number`, `account_number` 같은 전용 컬럼에 NOT NULL을 DB 레벨로 강제할 수 있는 건 JOINED뿐이다. SINGLE_TABLE은 이 검증을 전부 애플리케이션이 떠안는다.
- 서브타입이 2개뿐이라 조인 비용이 아직 작다. 조인 개수는 서브타입 수에 비례하므로, 지금 규모에서는 정규화 이득이 조인 비용보다 크다.

### 버린 안: SINGLE_TABLE

조회가 압도적으로 빈번해 조인 비용이 부담되거나, 무결성을 애플리케이션 검증만으로 충분히 감당할 수 있으면 더 적합해진다.

### 재평가 조건

- 서브타입 5개 이상으로 조인 개수가 부담될 때
- 운영 DB 실측 결과 JOINED 조인 비용이 무결성 이득을 상회할 때

## 근거형 질문

미션 가이드의 질문마다 최초 판단 → 연결한 코드·테스트·로그 → 검증 후 답변 순서로 적습니다.

1. 상속 매핑 전략을 선택할 때 조회 성능과 스키마 정규화를 어떻게 비교했나요?
- 판단: 조회 성능(조인 유무)과 정규화(nullable, NOT NULL 가능 여부)는 트레이드오프이며 어느 쪽도 절대 우월하지 않다.
- 근거: SINGLE_TABLE/JOINED DDL 비교, 다형성 조회 테스트 결과와 실제 SQL
- 답변: 서브타입이 적고(2개) 정합성이 중요해 정규화 쪽에 무게를 둬 JOINED를 선택. 서브타입이 많아지고 조회가 압도적으로 빈번해지면 판단이 바뀔 수 있다.

2. SINGLE_TABLE의 nullable 컬럼 비용과 JOINED의 조인 비용을 이번 모델에서 어떻게 확인했나요?
- 판단: nullable 비용은 DDL 컬럼 목록으로, 조인 비용은 SQL의 JOIN 개수로 직접 셀 수 있다.
- 근거: SINGLE_TABLE DDL의 nullable 컬럼 목록, JOINED 다형성 조회 SQL의 JOIN 개수
- 답변: 정성적 방향(있다/없다)은 로그로 확인했지만, 실제 ms 비용 차이는 H2 소규모 데이터로 측정하지 않았다 — 구조 확인에서 멈췄다.

3. 다형성 조회 SQL이 두 전략에서 어떻게 달랐고 H2 실험 결과를 어디까지 일반화할 수 있나요?
- 판단: SINGLE_TABLE은 단일 SELECT + discriminator 판별, JOINED는 서브타입 수만큼 LEFT OUTER JOIN 후 각 PK의 not-null로 타입 판별.
- 근거: 두 전략의 다형성 조회 SQL 로그
- 답변: 이론대로 확인. SQL 구조는 다른 DB 엔진에도 일반화 가능하지만, 옵티마이저가 다른 MySQL에서의 실제 응답시간까지는 일반화할 수 없다.

4. 두 매핑안 중 버린 안이 더 적합해지는 요구사항은 무엇인가요?
- 판단·답변: ADR "버린 안"·"재평가 조건" 참고 — 서브타입이 5개 이상으로 늘거나 다형성 조회가 트래픽 대부분을 차지하면 SINGLE_TABLE로 재평가.

## 리뷰 반영

자동 리뷰 수신 전
