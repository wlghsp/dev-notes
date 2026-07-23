# 함수형 데코레이터

함수(또는 람다)를 인자로 받아, 그 함수를 감싼 새 함수를 반환하는 패턴이다. 원본 함수의 코드는 안 건드리고 호출 앞뒤에 부가 로직(재시도, 타임아웃, 서킷 체크 등)을 끼워 넣는다는 목적은 spring-aop-proxy.md의 프록시와 같지만, 감싸는 대상이 클래스가 아니라 함수 하나라는 점이 다르다.

## Resilience4j에서의 사용

Resilience4j는 애노테이션 없이도 이 패턴을 직접 코드로 쓸 수 있게 해준다.

```java
Supplier<String> decorated = CircuitBreaker
    .decorateSupplier(circuitBreaker, () -> phiClient.getPhiAlarm(tagName));

String result = Try.ofSupplier(decorated)
    .recover(throwable -> fallbackValue)
    .get();
```

`decorateSupplier`가 원본 함수를 감싸서, 호출 시 서킷 상태를 체크하고 실패하면 예외를 던지는 새로운 함수를 돌려준다. 스프링 없이 순수 라이브러리만으로 쓸 수 있고, 데코레이터를 여러 겹 체이닝(타임리미터, 재시도 등을 겹겹이 두르는 것)할 수도 있다.

## 애노테이션 방식과의 관계

`@CircuitBreaker` 애노테이션은 이 함수형 데코레이터를 스프링이 프록시 뒤에 대신 만들어서 숨겨준 것이다. 애노테이션이 붙은 메서드가 호출되면, 스프링 내부적으로는 원본 메서드 호출을 함수형 데코레이터로 감싼 뒤 실행하는 것과 동일한 일이 일어난다.

즉 애노테이션 방식과 함수형 데코레이터 방식은 같은 메커니즘을 겉으로 다르게 노출한 것뿐이다. 애노테이션이 선언적이라면, 함수형 데코레이터는 그 뒤에서 실제로 일어나는 일을 코드로 직접 드러낸 명시적 버전이다.

참고: resilience4j.md
참고: spring-aop-proxy.md
