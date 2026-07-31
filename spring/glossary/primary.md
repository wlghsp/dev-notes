# @Primary

## 왜 필요한가

같은 타입의 빈이 여러 개 등록되어 있으면 타입만으로 자동 주입할 대상을 정할 수 없어 `NoUniqueBeanDefinitionException`이 터진다. `@Primary`는 이런 상황에서 "여러 후보 중 기본값은 이거다"라고 빈 정의 쪽에서 미리 선언해두는 방식이다.

```java
public interface PaymentService {}

@Primary
@Component
public class KakaoPayService implements PaymentService {}

@Component
public class TossPayService implements PaymentService {}
```

이렇게 해두면 `PaymentService` 타입으로 주입받는 자리에서 별다른 지정이 없어도 `KakaoPayService`가 선택된다.

## qualifier.md와의 차이

`@Primary`는 클래스(빈 정의) 쪽에서 "기본값은 나"라고 선언하는 방식이고, qualifier.md에서 다루는 `@Qualifier`는 주입받는 쪽에서 "이번엔 이걸 써라"라고 지정하는 방식이다. 관점이 반대다.

우선순위는 주입 지점의 명시적 지정이 항상 이긴다. `@Qualifier`가 있으면 그 이름으로 바로 결정되고, `@Qualifier`가 없을 때만 `@Primary`가 붙은 빈이 선택된다. 즉 `@Primary`는 "아무 지정도 없을 때의 기본값"이지, `@Qualifier`를 덮어쓰는 우선권은 아니다.

## 여러 개의 @Primary

같은 타입의 빈에 `@Primary`를 두 개 이상 붙이면 여전히 후보를 하나로 좁힐 수 없어서 예외가 난다. `@Primary`는 "후보 중 기본값 하나"를 정하는 용도이지, 여러 개를 붙여도 우선순위가 매겨지는 게 아니다.

## 언제 @Qualifier 대신 이걸 쓰는가

대부분의 주입 지점에서 같은 빈을 쓰고 극히 일부 예외적인 지점에서만 다른 빈을 써야 한다면, 흔히 쓰는 빈에 `@Primary`를 붙여 기본값으로 만들고 예외적인 지점에만 `@Qualifier`를 붙이는 조합이 자연스럽다. 반대로 어느 빈이 기본인지 애매하거나 호출부마다 의도를 명시적으로 드러내고 싶다면 `@Primary` 없이 모든 주입 지점에 `@Qualifier`를 쓰는 편이 더 명확하다.
