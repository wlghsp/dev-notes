# @Fallback

Spring Framework 6.2에 추가된 `org.springframework.context.annotation.Fallback` 어노테이션이다. primary.md에서 다루는 `@Primary`와 정반대 방향으로 동작하는 짝 개념이다.

## 하는 일

`@Primary`가 "여러 후보 중 이게 기본값이다"라고 우선권을 주는 방식이라면, `@Fallback`은 "다른 후보가 있으면 나는 뒤로 빠져라"라고 선언하는 방식이다.

```java
@Component
public class DefaultPaymentService implements PaymentService {}

@Fallback
@Component
public class BackupPaymentService implements PaymentService {}
```

같은 타입의 빈 중 `@Fallback`이 붙지 않은 후보가 하나라도 있으면 그쪽이 선택된다. `@Fallback`이 붙은 빈은 나머지 후보가 전부 `@Fallback`이거나 아예 없어서 유일하게 남을 때만 선택된다.

## 왜 필요한가 — @Primary로는 안 되는 경우

라이브러리가 기본 구현체를 제공하고, 사용자가 자기 구현체를 등록하면 그걸 우선시키고 싶은 상황을 생각해보자. 라이브러리 쪽 빈에 `@Primary`를 붙이면 사용자가 아무리 자기 빈을 등록해도 라이브러리 빈이 계속 이긴다. 반대로 아무 표시도 안 하면 사용자가 빈을 등록하는 순간 두 후보가 동시에 존재해 `NoUniqueBeanDefinitionException`이 터진다.

`@Fallback`은 이 문제를 뒤집어서 푼다. 라이브러리의 기본 구현체에 `@Fallback`을 붙여두면, 사용자가 자기 구현체를 등록하지 않았을 때는 라이브러리 빈이 유일한 후보라서 그대로 선택되고, 사용자가 자기 구현체를 등록하는 순간 `@Fallback`이 없는 그 빈이 우선 선택된다. "기본 구현체를 제공하되 사용자가 뭘 등록하면 자동으로 양보한다"는 동작이 별도 조건 분기 없이 어노테이션 하나로 해결된다.

## 주의할 점

`@Primary`, `@Fallback` 모두 "단일 주입 지점에서 후보를 하나로 좁힐 때"만 영향을 준다. `List<PaymentService>`나 `Map<String, PaymentService>`처럼 여러 개를 한꺼번에 주입받는 경우엔 `@Fallback`이 붙은 빈도 그대로 포함된다. 이 어노테이션이 빈을 숨기거나 컨테이너에서 제외하는 게 아니라, 단일 후보를 고르는 심사에서만 순위를 낮추는 것이라는 뜻이다.
