# Resilience4j fallback 메서드 계약

`@CircuitBreaker(fallbackMethod = "fallback")`을 쓸 때, fallback 메서드는 아무 시그니처나 허용되지 않는다. 정해진 규칙을 따라야 하고, 어기면 컴파일은 되지만 런타임에 예외가 난다.

## 계약 내용

fallback 메서드의 파라미터는 원본 메서드와 동일해야 하고, 그 뒤에 `Throwable`(또는 그 하위 타입)이 추가로 붙어야 한다.

```java
// 원본
public List<Item> getItems(String key) { ... }

// 올바른 fallback — 파라미터 동일 + 마지막에 Throwable
private List<Item> fallback(String key, Throwable t) { ... }
```

반환 타입도 원본과 호환돼야 한다.

## 왜 리플렉션으로 메서드를 찾는가

Resilience4j는 컴파일 시점이 아니라 런타임에 `fallbackMethod`에 지정한 문자열 이름으로 리플렉션을 통해 메서드를 찾는다. 문자열 이름이라 컴파일러가 오타나 시그니처 불일치를 잡아주지 못한다.

## 흔한 실수와 결과

이름을 잘못 쓰거나, 파라미터 타입을 하나라도 다르게 쓰거나, 마지막에 `Throwable`을 빠뜨리면 컴파일은 정상적으로 된다. 문제는 런타임에 실제로 fallback이 호출되는 시점, 즉 서킷이 열리거나 호출이 실패하는 시점에야 `NoSuchMethodException`으로 드러난다는 점이다. 평소 정상 동작할 때는 이 메서드가 아예 호출되지 않으므로, 장애 상황이 오기 전까지는 문제를 눈치채기 어렵다.

참고: self-invocation-and-spring-aop-proxy.md
