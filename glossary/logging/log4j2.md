# Log4j2

## 한 줄 정의

애플리케이션에서 발생하는 이벤트를 파일, 콘솔, 네트워크 등 원하는 곳에 기록하기 위한 Java 로깅 라이브러리.

## 왜 라이브러리가 필요한가

`System.out.println()`으로도 출력은 되지만, 이걸로는 아래 것들을 못 한다.

- 어느 클래스에서 찍혔는지 추적
- 로그 레벨(DEBUG / INFO / WARN / ERROR)로 필터링
- 운영 중에 출력 끄고 켜기
- 파일에 자동으로 날짜별로 저장

Log4j2는 이 문제를 해결하기 위한 라이브러리다.

## 핵심 구성 요소

**Logger** — 로그를 찍는 주체. 클래스마다 하나씩 만들어 쓴다.

```java
private static final Logger log = LogManager.getLogger(OrderService.class);
log.info("주문 생성: {}", orderId);
```

**Appender** — 로그를 어디에 보낼지 결정하는 출력 대상.
- ConsoleAppender: 콘솔
- FileAppender: 파일
- RollingFileAppender: 날짜/크기 기준으로 자동 분리되는 파일
- SocketAppender: 네트워크로 전송 (Logstash가 여기서 받음)

**Layout** — 로그를 어떤 형식으로 출력할지.
- PatternLayout: `[INFO] 2024-01-01 OrderService - 주문 생성: 123` 형태
- JsonLayout: JSON으로 직렬화 (ELK 스택과 연동 시 주로 사용)

**Filter** — 레벨이나 조건에 따라 출력 여부 결정.

## 로그가 남는 흐름

```
코드에서 log.info() 호출
    ↓
Logger가 레벨 확인 (현재 설정된 레벨보다 낮으면 버림)
    ↓
Filter 통과 여부 판단
    ↓
Appender로 전달
    ↓
Layout이 문자열로 포맷
    ↓
콘솔 / 파일 / 네트워크에 기록
```

## SLF4J와의 관계

Spring에서는 보통 `log.info()`를 SLF4J API로 호출한다.

SLF4J는 인터페이스(facade)이고, Log4j2는 그 구현체다.
덕분에 Log4j2를 Logback으로 교체해도 애플리케이션 코드는 바꾸지 않아도 된다.

```
코드 → SLF4J API → Log4j2 (또는 Logback, JUL 등)
```

Spring Boot 기본값은 Logback이다. Log4j2로 쓰려면 의존성을 교체해야 한다.

## Kafka / Logstash에서 쓰이는 이유

Kafka 브로커 자체가 JVM 위에서 돌기 때문에 내부 로깅을 Log4j2로 처리한다.
Logstash도 JVM 기반이라 동일하다.

2021년 Log4Shell(CVE-2021-44228) 취약점이 JVM 생태계 전반에 퍼진 이유가 이것이다.
Log4j2의 JNDI lookup 기능이 원격 코드 실행으로 이어질 수 있었고, Kafka/Logstash/Spring 모두 영향을 받았다.

## 참고

참고: logstash.md
