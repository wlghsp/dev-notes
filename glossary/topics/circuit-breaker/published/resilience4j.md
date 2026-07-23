# Resilience4j

Java용 장애 허용(fault tolerance) 라이브러리다. CircuitBreaker, Retry, RateLimiter, Bulkhead, TimeLimiter 같은 패턴을 어노테이션이나 함수형 데코레이터로 쉽게 붙일 수 있게 해준다.

## 왜 이런 라이브러리가 필요한가

circuit-breaker.md, retry.md, bulkhead.md, rate-limiter.md에서 다룬 패턴들은 개념적으로는 단순하지만, 직접 구현하려면 상태 관리(CLOSED/OPEN/HALF_OPEN 전이), 슬라이딩 윈도우 기반 실패율 계산, 스레드 안전성 같은 세부 사항을 전부 손으로 챙겨야 한다. Resilience4j는 이 구현을 라이브러리로 제공해서, 개발자는 설정값과 어노테이션만으로 이 패턴들을 적용할 수 있게 해준다.

## Netflix Hystrix와의 관계

Resilience4j 이전에는 Netflix Hystrix가 이 분야의 사실상 표준이었다. Hystrix가 유지보수 중단(maintenance mode)에 들어간 뒤, Resilience4j가 그 자리를 대체하는 라이브러리로 자리잡았다. Spring Cloud Circuit Breaker도 Hystrix 대신 Resilience4j를 기본 구현체로 채택하고 있다.

## 기본 사용 형태 (Spring 환경)

설정은 `application.yml`에 인스턴스 이름 단위로 작성하고, 코드에서는 어노테이션으로 그 이름을 참조한다.

```yaml
resilience4j:
  circuitbreaker:
    instances:
      example:
        sliding-window-size: 10
        failure-rate-threshold: 50
```

```java
@CircuitBreaker(name = "example", fallbackMethod = "fallback")
public Result call() { ... }
```

이 기본 형태를 Spring 환경에서 실제로 적용할 때 걸리는 함정들(프록시 문제, fallback 시그니처 규칙 등)은 spring-practice 폴더에서 다룬다.

참고: circuit-breaker.md
참고: bulkhead.md
참고: rate-limiter.md
참고: functional-decorator.md
