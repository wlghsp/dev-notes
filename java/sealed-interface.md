# sealed interface

Java 17에서 정식 도입된 문법으로, 이 interface를 구현(implement)할 수 있는 클래스나 interface를 명시적으로 제한한다. 일반 interface는 아무나 구현할 수 있지만, sealed interface는 작성자가 허용한 타입만 구현할 수 있다.

```java
public sealed interface Shape
    permits Circle, Square, Triangle {
}

public final class Circle implements Shape { ... }
public final class Square implements Shape { ... }
public final class Triangle implements Shape { ... }
```

## 기존 interface와 다른 점

**구현 범위를 제어할 수 있는가**
일반 interface는 어떤 클래스든 `implements`만 하면 구현체가 될 수 있다. 외부 라이브러리를 쓰는 코드가 그 interface를 구현해서 새 타입을 추가하는 것도 막을 방법이 없다. sealed interface는 `permits` 절에 나열된 타입만 구현을 허용한다. 나열되지 않은 클래스가 구현을 시도하면 컴파일 에러가 난다.

**구현체 목록을 컴파일러가 아는가**
일반 interface는 구현체가 몇 개이고 무엇인지 컴파일러가 알 수 없다. 런타임에 어떤 클래스가 추가로 나타날지 알 수 없기 때문이다. sealed interface는 `permits`에 적힌 타입이 전부라는 것을 컴파일러가 보장한다. 이 "전부 안다"는 특성이 아래 switch exhaustiveness의 기반이 된다.

**switch문에서 exhaustiveness(완전성) 검사가 가능한가**
일반 interface를 switch로 분기하면, 컴파일러는 모든 구현체를 다 다뤘는지 확인할 방법이 없다. 그래서 항상 `default` 분기가 필요하고, 새 구현체가 추가돼도 컴파일러가 경고해주지 않는다. sealed interface는 구현체가 `permits`로 고정되어 있으므로, switch문이 모든 구현체를 다루면 `default` 없이도 컴파일이 통과한다. 반대로 새 구현체를 permits에 추가했는데 기존 switch문에서 그 케이스를 빠뜨리면 컴파일 에러가 난다.

```java
// sealed interface + switch: default 불필요, 케이스 누락 시 컴파일 에러
double area = switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Square s -> s.side() * s.side();
    case Triangle t -> 0.5 * t.base() * t.height();
};
```

## permits와 구현체 선언 방식

`permits` 절은 같은 파일 안에 구현체가 모두 있으면 생략할 수 있다. 컴파일러가 파일 안의 구현체를 자동으로 인식한다.

sealed interface를 구현하는 타입은 세 가지 중 하나를 명시해야 한다.
- `final` — 더 이상 상속/구현을 허용하지 않는다.
- `sealed` — 자신도 sealed로 남아 permits로 제한된 하위 타입만 허용한다.
- `non-sealed` — 다시 제한을 풀어서 누구나 구현할 수 있게 한다.

이 셋 중 하나를 반드시 골라야 하는 이유는, sealed interface가 보장하는 "구현체를 전부 안다"는 특성이 하위 단계에서 조용히 깨지는 것을 막기 위해서다.

## enum과 무엇이 다른가 — 상태별로 다른 데이터를 가질 때

"상태 값 하나"만 표현한다면 enum으로 충분하다. 하지만 상태마다 함께 있어야 하는 데이터가 다르다면 enum은 그 규칙을 코드로 강제하지 못한다.

주문 상태를 예로 들면, 대기(Pending)/승인(Approved)/실패(Failed) 세 상태가 있고 승인 상태에만 거래키(transactionKey)가, 실패 상태에만 실패 사유(reason)가 있어야 한다고 하자. enum으로 표현하면 이렇게 된다.

```java
public enum OrderStatus { PENDING, APPROVED, FAILED }

private OrderStatus status;      // 셋 중 하나
private String transactionKey;   // APPROVED일 때만 값 있음 (아니면 null)
private String reason;           // FAILED일 때만 값 있음 (아니면 null)
```

문제는 "APPROVED면 transactionKey가 반드시 있어야 한다"는 규칙이 코드 어디에도 강제되어 있지 않다는 것이다. 컴파일러는 `status = APPROVED`인데 `transactionKey = null`인 상태를 막을 방법이 없다. 사람이 실수로 값 채우는 걸 빼먹으면 런타임에야 문제가 드러난다.

sealed interface와 record를 합치면 상태와 그 상태에 딸린 데이터를 하나의 타입으로 묶을 수 있다.

```java
public sealed interface OrderStatus {
    record Pending() implements OrderStatus {}
    record Approved(String transactionKey) implements OrderStatus {}
    record Failed(String reason) implements OrderStatus {}
}
```

`Approved`라는 타입 자체가 `transactionKey` 필드를 갖고 태어난다. `Approved` 인스턴스를 만들려면 무조건 `transactionKey`를 줘야 하므로, "승인인데 거래키가 없는" 상태 자체가 애초에 만들어질 수 없다. `Pending`엔 그런 필드가 아예 없고, `Failed`는 `reason`만 가진다. 잘못된 조합을 타입 구조 자체가 막는다.

switch 분기에서도 차이가 드러난다. enum switch에서 `FAILED` 케이스 처리를 깜빡해도 컴파일은 통과되고 런타임에 조용히 넘어간다. sealed interface는 케이스 하나를 빠뜨리면 컴파일 자체가 안 된다.

```java
switch (status) {
    case OrderStatus.Pending p -> ...;
    case OrderStatus.Approved a -> ...; // a.transactionKey()로 바로 꺼내 씀, null 체크 불필요
    // Failed 케이스를 빠뜨리면 컴파일 에러
}
```

다만 DB에 저장할 때는 주의가 필요하다. JPA 같은 ORM은 컬럼 값으로 sealed interface를 직접 받지 못하므로, "DB 저장용 순수 enum"과 "도메인 로직용 sealed interface"를 분리해서 각자 잘하는 역할만 맡기는 방식이 실무에서 쓰인다.

## 언제 쓰는가

구현체 종류가 정해져 있고 앞으로도 그 종류 안에서만 분기 처리를 하고 싶을 때 적합하다. 예를 들어 도형(Circle/Square/Triangle), 결제 수단(CardPayment/BankTransfer/CashPayment)처럼 "이게 전부"라고 확신할 수 있는 도메인 모델링에 쓰인다. 반대로 라이브러리를 배포해서 사용자가 자유롭게 구현체를 추가하도록 열어두고 싶다면 sealed는 맞지 않고 일반 interface가 적합하다.
