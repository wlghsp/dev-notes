# Discriminator Column

SINGLE_TABLE 상속 매핑 전략(참고: inheritance-mapping-strategy.md)에서 서브타입을 구분하기 위해 쓰는 컬럼이다.

## 왜 필요한가

SINGLE_TABLE은 부모와 모든 자식의 데이터가 테이블 하나에 몰려 있다. 그래서 row 하나만 봐서는 이게 어떤 자바 클래스(예: `CardPayment`인지 `BankTransferPayment`인지)로 매핑되어야 하는지 알 방법이 없다.

컬럼 값이 채워져 있는지(예: `card_number`가 NULL이 아닌지)로 타입을 추론하는 방법도 떠올릴 수 있지만, 이건 우연히 값이 겹치거나 비어있는 경우에 취약하고 JPA 스펙상 신뢰할 수 있는 구분 방식이 아니다.

## 동작 방식

부모 클래스에 `@DiscriminatorColumn`을 선언하면, 그 컬럼(기본 이름은 `DTYPE`)에 각 row가 어떤 타입인지 나타내는 문자열이 저장된다. 조회 시 Hibernate는 이 컬럼 값을 읽어서 어떤 자바 클래스로 매핑할지 결정한다.

각 자식 클래스는 `@DiscriminatorValue`로 자신이 이 컬럼에 어떤 값을 쓸지 지정한다. 지정하지 않으면 기본적으로 클래스명이 그 값으로 쓰인다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type")
public abstract class Payment { ... }

@Entity
@DiscriminatorValue("CARD")
public class CardPayment extends Payment { ... }

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPayment extends Payment { ... }
```

이렇게 저장된 row는 `payment_type` 컬럼에 `"CARD"` 또는 `"BANK_TRANSFER"`라는 값을 갖게 되고, 부모 타입(`Payment`)으로 다형성 조회를 하면 이 값을 보고 실제 자바 객체를 `CardPayment` 또는 `BankTransferPayment`로 복원한다.

## JOINED에서는 왜 필수가 아닌가

JOINED 전략에서는 자식 테이블 자체가 부모의 PK를 공유하는 FK를 갖고 있다. 즉 "이 자식 테이블에 매칭되는 row가 있다"는 사실 자체가 이미 타입 정보를 담고 있어서, 별도의 discriminator 컬럼 없이도 타입을 구분할 수 있다. 다만 Hibernate 구현에 따라 JOINED에서도 선택적으로 쓸 수는 있다.
