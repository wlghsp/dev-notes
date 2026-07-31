# Fallback (fallbackMethod)

Resilience4j를 사용하는 Spring Cloud Circuit Breaker에서 `@CircuitBreaker` 어노테이션에 딸린 속성이다. 호출이 실패했을 때 대신 실행할 메서드를 지정한다.

## 기본 사용법

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
public PaymentResult pay(Order order) {
    return paymentClient.pay(order);
}

public PaymentResult fallback(Order order, Throwable t) {
    return PaymentResult.failed();
}
```

원본 메서드가 예외를 던지거나 서킷이 open 상태여서 호출 자체가 차단되면, 스프링은 원본 메서드 대신 `fallbackMethod`에 지정된 메서드를 실행한다.

## 시그니처 규칙

fallback 메서드는 원본 메서드와 파라미터가 같아야 하고, 마지막에 예외 타입 파라미터(`Throwable` 또는 그 하위 타입)를 하나 추가로 받는다. 반환 타입도 원본과 같아야 한다.

이 예외 파라미터로 어떤 이유로 실패했는지(타임아웃인지, 서킷 open인지, 실제 예외인지)를 구분해서 처리할 수 있다. 파라미터 타입을 `CallNotPermittedException`처럼 구체적으로 좁히면 그 예외에만 반응하는 fallback 메서드를 따로 만들 수도 있다 — 같은 이름으로 여러 개를 오버로드해서 예외 타입별로 분기하는 방식이다.

## 언제 호출되는가

원본 메서드 실행 중 예외가 발생했을 때, 그리고 서킷브레이커가 open 상태라 원본 메서드 호출 자체가 차단됐을 때(`CallNotPermittedException`) 둘 다 fallback으로 이어진다. 즉 fallback은 "원본이 실패한 경우"뿐 아니라 "원본을 아예 시도하지 못한 경우"까지 포함해서 실행된다.

## fallback을 어떻게 작성할지는 별개의 문제

`fallbackMethod`는 실패 시 어떤 메서드가 실행되는지에 대한 매커니즘일 뿐이고, 그 안에서 예외를 삼킬지 다시 던질지는 호출의 성격(읽기냐 쓰기냐)에 따라 갈리는 설계 판단이다. 이 판단 기준은 circuit-breaker-fallback-design.md에 정리했다.
