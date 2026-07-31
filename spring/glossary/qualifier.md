# @Qualifier

## 왜 필요한가

`@Autowired`는 기본적으로 타입 기준으로 빈을 찾는다. 같은 타입의 빈이 여러 개 등록되어 있으면 스프링이 어느 걸 주입해야 할지 판단할 수 없어서 `NoUniqueBeanDefinitionException`이 터진다.

```java
public interface PaymentService {}

@Component
public class KakaoPayService implements PaymentService {}

@Component
public class TossPayService implements PaymentService {}
```

이 상태에서 `PaymentService` 타입으로 주입받으려 하면 두 후보 중 무엇을 골라야 할지 스프링이 알 수 없다. `@Qualifier`는 이름으로 후보를 하나로 좁히는 역할을 한다.

## 사용 방식

```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(@Qualifier("kakaoPayService") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

기본적으로 빈 이름은 클래스명의 첫 글자를 소문자로 바꾼 것(`kakaoPayService`)이 자동으로 등록된다. `@Qualifier`에 이 이름을 지정하면 타입이 같은 후보들 중 해당 이름의 빈만 골라 주입한다.

`@Component("myPay")`처럼 빈 이름을 직접 지정해뒀다면 `@Qualifier`에도 그 이름을 그대로 써야 한다.

## 우선순위 판단 순서

스프링이 같은 타입의 빈 여러 개를 마주쳤을 때 후보를 좁히는 순서는 이렇다.

먼저 `@Qualifier`가 명시되어 있으면 그 이름으로 바로 결정한다. `@Qualifier`가 없고 primary.md에서 다루는 `@Primary`가 붙은 빈이 후보 중에 있으면 그걸 선택한다. 둘 다 없으면 필드/파라미터 이름과 빈 이름이 일치하는지를 마지막으로 본다. 이마저 안 맞으면 예외가 터진다.

`@Primary`와 `@Qualifier`가 동시에 걸려 있어 충돌하면(`@Primary` 빈이 있어도 다른 빈을 `@Qualifier`로 명시하면) `@Qualifier`가 이긴다. 주입 지점에서의 명시적 지정이 클래스 쪽 기본값 선언보다 우선하기 때문이다.

## 커스텀 Qualifier

문자열 이름 대신 의미 있는 어노테이션을 만들어 쓸 수도 있다.

```java
@Qualifier
@Retention(RUNTIME)
public @interface KakaoPay {}

@Component
@KakaoPay
public class KakaoPayService implements PaymentService {}

public OrderService(@KakaoPay PaymentService paymentService) { ... }
```

문자열은 오타가 나도 컴파일 시점에 안 잡히는데, 커스텀 어노테이션은 타입이라서 오타 자체가 불가능하다. 스프링 프레임워크 내부에서도 `@Qualifier`를 메타 어노테이션으로 활용해 `@Profile` 등 여러 커스텀 어노테이션을 만드는 방식이 이것이다.
