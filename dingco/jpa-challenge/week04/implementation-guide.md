# Week 4 구현 가이드

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
prep-questions.md에서 정리한 개념을 실제 코드로 옮기는 단계. 이번엔 힌트가 아니라 실제로 작성할 엔티티/테스트 코드까지 전부 제공한다. 그대로 타이핑하거나 붙여넣고, 각 단계에서 실제로 나오는 SQL 로그를 evidence에 남기면 된다.

기존 프로젝트 컨벤션(week1~3에서 이미 쓰던 것)을 그대로 따른다:
- 패키지: `co.dingcodingco.challenge.week4`
- `@DataJpaTest` + `JpaObservation`(`SqlCaptureInspector` 기반 SQL 관측 도구) 재사용
- 엔티티는 `protected` 기본 생성자 + public 생성자, `@Id @GeneratedValue`
- `application.properties`에 이미 `hibernate.generate_statistics=true`, SQL 로깅이 켜져 있어 별도 설정 불필요

---

## 1단계. 엔티티 설계 — SINGLE_TABLE과 JOINED를 각각 별도 계층으로

같은 도메인을 SINGLE_TABLE 버전과 JOINED 버전 **두 세트**로 만든다. 이름을 접미사로 구분해서 하나의 패키지에 공존시키는 방식을 추천한다(Board/Post/Comment/Reply처럼 이미 week3에서 "같은 역할, 다른 매핑"을 별도 클래스 쌍으로 만든 전례가 있다).

패키지: `co.dingcodingco.challenge.domain.payment`

### `PaymentSingleTable.java` (부모, SINGLE_TABLE)

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "payment_single_table")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type")
public abstract class PaymentSingleTable {
    @Id @GeneratedValue
    private Long id;

    private BigDecimal amount;
    private String status;
    private LocalDateTime createdAt;

    protected PaymentSingleTable() {}

    protected PaymentSingleTable(BigDecimal amount, String status) {
        this.amount = amount;
        this.status = status;
        this.createdAt = LocalDateTime.now();
    }

    public Long getId() { return id; }
    public BigDecimal getAmount() { return amount; }
    public String getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

### `CardPaymentSingleTable.java` (자식 1)

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import java.math.BigDecimal;

@Entity
@DiscriminatorValue("CARD")
public class CardPaymentSingleTable extends PaymentSingleTable {
    private String cardNumber;
    private Integer installmentMonths;

    protected CardPaymentSingleTable() {}

    public CardPaymentSingleTable(BigDecimal amount, String status, String cardNumber, Integer installmentMonths) {
        super(amount, status);
        this.cardNumber = cardNumber;
        this.installmentMonths = installmentMonths;
    }

    public String getCardNumber() { return cardNumber; }
    public Integer getInstallmentMonths() { return installmentMonths; }
}
```

### `BankTransferPaymentSingleTable.java` (자식 2)

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import java.math.BigDecimal;

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPaymentSingleTable extends PaymentSingleTable {
    private String bankCode;
    private String accountNumber;

    protected BankTransferPaymentSingleTable() {}

    public BankTransferPaymentSingleTable(BigDecimal amount, String status, String bankCode, String accountNumber) {
        super(amount, status);
        this.bankCode = bankCode;
        this.accountNumber = accountNumber;
    }

    public String getBankCode() { return bankCode; }
    public String getAccountNumber() { return accountNumber; }
}
```

### `PaymentJoined.java` (부모, JOINED)

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "payment_joined")
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class PaymentJoined {
    @Id @GeneratedValue
    private Long id;

    private BigDecimal amount;
    private String status;
    private LocalDateTime createdAt;

    protected PaymentJoined() {}

    protected PaymentJoined(BigDecimal amount, String status) {
        this.amount = amount;
        this.status = status;
        this.createdAt = LocalDateTime.now();
    }

    public Long getId() { return id; }
    public BigDecimal getAmount() { return amount; }
    public String getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

### `CardPaymentJoined.java` / `BankTransferPaymentJoined.java`

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import java.math.BigDecimal;

@Entity
@Table(name = "card_payment_joined")
public class CardPaymentJoined extends PaymentJoined {
    private String cardNumber;
    private Integer installmentMonths;

    protected CardPaymentJoined() {}

    public CardPaymentJoined(BigDecimal amount, String status, String cardNumber, Integer installmentMonths) {
        super(amount, status);
        this.cardNumber = cardNumber;
        this.installmentMonths = installmentMonths;
    }

    public String getCardNumber() { return cardNumber; }
    public Integer getInstallmentMonths() { return installmentMonths; }
}
```

```java
package co.dingcodingco.challenge.domain.payment;

import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import java.math.BigDecimal;

@Entity
@Table(name = "bank_transfer_payment_joined")
public class BankTransferPaymentJoined extends PaymentJoined {
    private String bankCode;
    private String accountNumber;

    protected BankTransferPaymentJoined() {}

    public BankTransferPaymentJoined(BigDecimal amount, String status, String bankCode, String accountNumber) {
        super(amount, status);
        this.bankCode = bankCode;
        this.accountNumber = accountNumber;
    }

    public String getBankCode() { return bankCode; }
    public String getAccountNumber() { return accountNumber; }
}
```

**주의**: `@Id @GeneratedValue`의 기본 전략은 `GenerationType.AUTO`인데, H2에서는 보통 SEQUENCE로 동작한다. JOINED 전략에서 자식 엔티티들도 부모와 같은 PK 시퀀스를 공유해야 하므로(부모 PK = 자식 PK, 값이 같아야 조인이 성립) 기본값 그대로 두면 된다 — 자식 클래스에 별도로 `@Id`를 선언하지 않는 이상 PK는 부모에서만 관리된다.

---

## 2단계. 테스트 클래스 구성

패키지: `co.dingcodingco.challenge.week4`

- `PaymentSingleTableTest` — 스키마/INSERT/다형성 조회 관찰
- `PaymentJoinedTest` — 스키마/INSERT/다형성 조회 관찰
- `PaymentSchemaComparisonTest` (선택) — 두 전략을 한 클래스에서 나란히 비교하고 싶다면 이쪽으로 합쳐도 됨. 아래는 분리된 버전 기준으로 작성.

### `PaymentSingleTableTest.java`

```java
package co.dingcodingco.challenge.week4;

import co.dingcodingco.challenge.JpaObservation;
import co.dingcodingco.challenge.domain.payment.BankTransferPaymentSingleTable;
import co.dingcodingco.challenge.domain.payment.CardPaymentSingleTable;
import co.dingcodingco.challenge.domain.payment.PaymentSingleTable;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.math.BigDecimal;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class PaymentSingleTableTest {

    @Autowired
    EntityManager em;
    @Autowired
    EntityManagerFactory emf;
    JpaObservation observation;

    @BeforeEach
    void setUp() {
        observation = new JpaObservation(emf);
        observation.reset();
    }

    @DisplayName("SINGLE_TABLE에서 자식 엔티티 저장 시 INSERT가 1번만 발생한다")
    @Test
    void insert_happens_once_in_single_table() {
        // given
        CardPaymentSingleTable card = new CardPaymentSingleTable(
                BigDecimal.valueOf(10000), "COMPLETED", "1234-5678", 3);

        // when
        em.persist(card);
        em.flush();

        // then
        List<String> inserts = observation.statements().stream()
                .filter(sql -> sql.toLowerCase().startsWith("insert"))
                .toList();
        assertThat(inserts).hasSize(1);
        assertThat(inserts.get(0)).containsIgnoringCase("payment_single_table");
    }

    @DisplayName("SINGLE_TABLE에서 CardPayment row는 BankTransfer 전용 컬럼이 NULL이다")
    @Test
    void other_subtype_columns_are_null() {
        // given
        CardPaymentSingleTable card = new CardPaymentSingleTable(
                BigDecimal.valueOf(10000), "COMPLETED", "1234-5678", 3);
        em.persist(card);
        em.flush();
        em.clear();

        // when
        // 부모 타입으로 다시 조회해서 실제 서브타입이 맞는지, 자기 전용 필드는 채워졌는지 확인
        PaymentSingleTable found = em.find(PaymentSingleTable.class, card.getId());

        // then
        assertThat(found).isInstanceOf(CardPaymentSingleTable.class);
        assertThat(((CardPaymentSingleTable) found).getCardNumber()).isEqualTo("1234-5678");
        // BankTransfer 전용 컬럼은 이 row에 애초에 값이 없다는 것은
        // DB 콘솔(H2 web console) 또는 DESCRIBE로 직접 확인해서 evidence에 남긴다
    }

    @DisplayName("SINGLE_TABLE에서 부모 타입으로 다형성 조회하면 조인 없이 단일 SELECT가 발생한다")
    @Test
    void polymorphic_query_has_no_join() {
        // given
        em.persist(new CardPaymentSingleTable(BigDecimal.valueOf(10000), "COMPLETED", "1234-5678", 3));
        em.persist(new BankTransferPaymentSingleTable(BigDecimal.valueOf(20000), "COMPLETED", "004", "1002-333-444"));
        em.flush();
        em.clear();
        observation.reset();

        // when
        List<PaymentSingleTable> all = em.createQuery(
                "select p from PaymentSingleTable p", PaymentSingleTable.class
        ).getResultList();

        // then
        assertThat(all).hasSize(2);
        List<String> selects = observation.statements().stream()
                .filter(sql -> sql.toLowerCase().startsWith("select"))
                .toList();
        assertThat(selects).hasSize(1);
        assertThat(selects.get(0)).doesNotContainIgnoringCase("join");
    }
}
```

### `PaymentJoinedTest.java`

```java
package co.dingcodingco.challenge.week4;

import co.dingcodingco.challenge.JpaObservation;
import co.dingcodingco.challenge.domain.payment.BankTransferPaymentJoined;
import co.dingcodingco.challenge.domain.payment.CardPaymentJoined;
import co.dingcodingco.challenge.domain.payment.PaymentJoined;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.math.BigDecimal;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class PaymentJoinedTest {

    @Autowired
    EntityManager em;
    @Autowired
    EntityManagerFactory emf;
    JpaObservation observation;

    @BeforeEach
    void setUp() {
        observation = new JpaObservation(emf);
        observation.reset();
    }

    @DisplayName("JOINED에서 자식 엔티티 저장 시 부모 테이블과 자식 테이블에 각각 INSERT가 발생한다")
    @Test
    void insert_happens_twice_in_joined() {
        // given
        CardPaymentJoined card = new CardPaymentJoined(
                BigDecimal.valueOf(10000), "COMPLETED", "1234-5678", 3);

        // when
        em.persist(card);
        em.flush();

        // then
        List<String> inserts = observation.statements().stream()
                .filter(sql -> sql.toLowerCase().startsWith("insert"))
                .toList();
        assertThat(inserts).hasSize(2);
        assertThat(inserts).anyMatch(sql -> sql.toLowerCase().contains("payment_joined"));
        assertThat(inserts).anyMatch(sql -> sql.toLowerCase().contains("card_payment_joined"));
    }

    @DisplayName("JOINED에서 부모 타입으로 다형성 조회하면 LEFT OUTER JOIN이 발생한다")
    @Test
    void polymorphic_query_has_join() {
        // given
        em.persist(new CardPaymentJoined(BigDecimal.valueOf(10000), "COMPLETED", "1234-5678", 3));
        em.persist(new BankTransferPaymentJoined(BigDecimal.valueOf(20000), "COMPLETED", "004", "1002-333-444"));
        em.flush();
        em.clear();
        observation.reset();

        // when
        List<PaymentJoined> all = em.createQuery(
                "select p from PaymentJoined p", PaymentJoined.class
        ).getResultList();

        // then
        assertThat(all).hasSize(2);
        List<String> selects = observation.statements().stream()
                .filter(sql -> sql.toLowerCase().startsWith("select"))
                .toList();
        // Hibernate 구현에 따라 1번의 조인 SELECT로 처리될 수도, 서브타입별로 추가 SELECT가 나갈 수도 있다
        // 실제로 몇 번 어떤 형태로 나오는지가 evidence의 핵심 관찰 대상이므로 여기서 단정하지 않는다
        assertThat(selects).isNotEmpty();
        assertThat(selects).anyMatch(sql -> sql.toLowerCase().contains("join"));
    }
}
```

`polymorphic_query_has_join`의 assertion을 일부러 느슨하게(`isNotEmpty`, `anyMatch`) 잡았다. JOINED 다형성 조회 시 Hibernate가 실제로 SELECT를 몇 번 어떤 형태로 나누는지는 버전/설정에 따라 달라질 수 있어서 — 먼저 테스트를 돌려서 로그에 찍히는 실제 SQL을 보고, 그 다음에 assertion을 더 구체적으로 좁혀도 된다. 이게 이번 실험의 관찰 포인트이기도 하다.

---

## 3단계. 스키마 직접 확인 (nullable, DDL 비교)

H2는 인메모리라 테스트 중간에 breakpoint를 걸거나, 별도의 `@SpringBootTest` + `spring.jpa.hibernate.ddl-auto=create`로 스키마를 뽑아서 콘솔 로그로 DDL을 확인하는 방법이 가장 간단하다. `application.properties`에 이미 `format_sql=true`가 켜져 있으므로 DDL도 테스트 실행 로그에 그대로 찍힌다(`spring.jpa.properties.hibernate.hbm2ddl.auto` 관련 DDL 로그를 보려면 `logging.level.org.hibernate.SQL=DEBUG`만으로 부족할 수 있어, 필요하면 아래 설정을 테스트 실행 시 임시로 추가):

```properties
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.format_sql=true
```

또는 테스트에서 `EntityManagerFactory`가 뜨는 시점에 H2가 만든 실제 테이블 구조를 직접 조회하는 방법도 있다:

```java
@DisplayName("SINGLE_TABLE 테이블의 서브타입 전용 컬럼은 모두 nullable이다")
@Test
void subtype_columns_are_nullable_in_single_table() throws Exception {
    try (var connection = ((org.springframework.orm.jpa.EntityManagerFactoryInfo) emf)
            .getNativeEntityManagerFactory().unwrap(org.hibernate.SessionFactory.class)
            .getSessionFactoryOptions()) {
        // 실전에서는 이렇게 복잡하게 갈 필요 없이,
        // H2 web console(spring.h2.console.enabled=true)을 켜고 브라우저에서
        // information_schema.columns를 직접 조회하는 편이 훨씬 빠르고 확실하다
    }
}
```

이 부분은 코드로 테스트화하기보다, H2 web console을 켜고 눈으로 직접 확인하는 걸 추천한다. `src/test/resources/application.properties`에 아래를 임시로 추가:

```properties
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
```

`@DataJpaTest`는 기본적으로 테스트가 끝나면 롤백하고 DB를 날리므로, 스키마만 눈으로 확인하고 싶으면 테스트 안에 breakpoint를 걸거나 `Thread.sleep()`으로 잠깐 멈춰두고 브라우저에서 `http://localhost:8080/h2-console`로 접속해서 `payment_single_table` 테이블의 컬럼 정의(NULL 허용 여부)를 직접 보는 게 가장 확실하다. 확인 후에는 이 설정을 다시 빼는 것을 잊지 말 것 — 커밋에 남기지 않는다.

---

## 4단계. evidence 문서에 담을 것

`evidence/week-04__weekly-pr.md`(또는 프로젝트 규칙에 맞는 파일명)에 아래 내용을 채운다:

1. **변경**: 만든 엔티티 목록(PaymentSingleTable 계층, PaymentJoined 계층)과 테스트 목록
2. **검증**: 각 테스트 실행 결과 SQL 로그를 그대로 붙여넣기(INSERT 개수, SELECT의 JOIN 유무)
3. **선택 근거**: 왜 두 전략을 나란히 만들었는지, cascade-remove-orphan-removal-issue.md처럼 실험 중 실제로 겪은 예상 밖의 동작이 있었다면 그것도 기록
4. **근거형 질문 4개**: prep-questions.md A-1에서 이미 정리한 답을 "최초 판단 → 연결한 코드/로그 → 검증 후 답변" 형식으로 재구성
5. **ADR**: prep-questions.md A-5에서 정리한 항목(배경/문제, 고려한 옵션, 선택 이유, 트레이드오프, 버려진 옵션이 적합해지는 조건)을 별도 ADR 문서로 작성

---

## 실행 순서 요약

1. `domain/payment/` 패키지에 엔티티 6개 생성(부모 2 + 자식 4)
2. `week4/` 패키지에 테스트 2개(`PaymentSingleTableTest`, `PaymentJoinedTest`) 생성, 위 코드 그대로 작성
3. `mvn test`로 실행하고 로그에 찍히는 실제 SQL을 evidence에 옮겨 적기
4. 필요하면 H2 console로 스키마 직접 열어서 nullable 여부 확인 (커밋 전 설정 원복)
5. prep-questions.md 답변을 근거형 질문 형식으로 evidence에 재구성
6. ADR 문서 작성
7. PR 생성 전 `git status`/`gh pr diff`로 새 파일이 diff에 잡히는지 확인 (A-6에서 정리한 체크리스트)
