# SLF4J

## 한 줄 정의

Java 로깅 라이브러리들을 갈아끼울 수 있도록 만든 추상 인터페이스 레이어.

## 왜 존재하는가

Log4j, Logback, Log4j2, JUL(java.util.logging) 등 로깅 구현체가 여러 개다.
라이브러리마다 API가 달라서, 구현체를 바꾸면 호출 코드를 전부 수정해야 했다.

SLF4J는 이 문제를 facade 패턴으로 해결한다.
코드는 SLF4J API만 호출하고, 실제 로깅은 바인딩된 구현체가 처리한다.

```
애플리케이션 코드
    ↓ (SLF4J API)
slf4j-api.jar
    ↓ (바인딩)
Logback / Log4j2 / JUL 중 하나
```

## 사용법

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(OrderService.class);
log.info("주문 생성: {}", orderId);
```

코드 어디에도 Logback이나 Log4j2가 등장하지 않는다.
구현체는 classpath에 뭐가 있느냐로 결정된다.

## 바인딩 구조

SLF4J가 구현체와 연결되려면 바인딩 jar가 필요하다.

- Logback 쓸 때: `logback-classic`이 SLF4J 바인딩을 내장하고 있음
- Log4j2 쓸 때: `log4j-slf4j2-impl` 추가 필요
- JUL 쓸 때: `slf4j-jdk14` 추가 필요

바인딩이 2개 이상 classpath에 있으면 SLF4J가 경고를 띄우고 하나만 선택한다.

## Spring Boot 기본 구성

Spring Boot는 기본으로 Logback을 쓴다.
`spring-boot-starter`에 `spring-boot-starter-logging`이 포함되어 있고,
그 안에 `logback-classic`이 들어있다.

Log4j2로 바꾸려면 기본 로깅 의존성을 제거하고 교체해야 한다.

```
spring-boot-starter-logging 제거
    ↓
spring-boot-starter-log4j2 추가
```

## SLF4J가 없다면

서드파티 라이브러리 A는 Log4j2를 직접 쓰고,
라이브러리 B는 Logback을 직접 쓴다면,
같은 애플리케이션에서 두 구현체가 동시에 뜨는 상황이 생긴다.

SLF4J를 쓰면 라이브러리들이 모두 SLF4J API만 호출하고,
최종 구현체는 애플리케이션이 하나로 통일해서 제어할 수 있다.

## 참고

참고: log4j2.md
참고: logback.md
