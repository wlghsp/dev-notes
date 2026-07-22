# Feign 인터페이스에 AOP 어노테이션이 안 먹는 이유

`@CircuitBreaker` 같은 AOP 어노테이션을 OpenFeign 클라이언트 인터페이스의 메서드에 직접 붙이면 동작하지 않는다. self-invocation 문제와는 원인이 다른, Feign 고유의 제약이다.

## 왜 안 되는가

Feign은 인터페이스만 정의하면 그 구현체(프록시)를 런타임에 동적으로 생성해준다. 이 프록시는 Feign 자체의 목적(HTTP 요청 매핑)을 위해 만들어진 것이라, Spring AOP가 별도로 감쌀 자리가 없다. Spring이 어노테이션을 처리하려면 자신의 AOP 프록시가 메서드 호출을 가로챌 수 있어야 하는데, Feign 인터페이스 메서드 호출은 이미 Feign 자체의 프록시로 처리되고 끝나버린다.

```mermaid
flowchart LR
    Caller["호출부"] --> FeignProxy["Feign이 만든 프록시<br/>(HTTP 요청 매핑 담당)"]
    FeignProxy --> HTTP["실제 HTTP 호출"]
    FeignProxy -.->|"Spring AOP가<br/>끼어들 자리 없음"| HTTP
```

## 해결 방법 — 호출하는 쪽에 어노테이션을 붙인다

Feign 인터페이스 자체가 아니라, 그 Feign 클라이언트를 호출하는 별도 컴포넌트의 메서드에 `@CircuitBreaker`를 붙인다. 이 컴포넌트는 일반 스프링 빈이므로 정상적으로 AOP 프록시의 대상이 된다.

```java
@Component
public class ExternalLookupService {

    private final SomeFeignClient client;

    @CircuitBreaker(name = "someClient", fallbackMethod = "fallback")
    public Result lookup(String key) {
        return client.lookup(key); // Feign 호출은 이 메서드 안에서
    }

    private Result fallback(String key, Throwable t) {
        return Result.empty();
    }
}
```

이 패턴은 self-invocation 문제의 해결책(별도 컴포넌트로 분리)과 결과적으로 같은 모양이 된다. 다만 원인은 다르다 — self-invocation은 같은 클래스 안에서 호출해서 프록시를 못 거치는 문제고, 이건 애초에 Feign 프록시가 Spring AOP 프록시와 다른 레이어라 어노테이션이 인터페이스 메서드에 붙어도 처리할 주체가 없는 문제다.

참고: self-invocation-and-spring-aop-proxy.md
참고: resilience4j-fallback-method-contract.md
