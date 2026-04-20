# Spring이 편리하게 숨겨버린 것들 — 시리즈 목차

> Spring은 강력하다. 너무 강력해서, 개발자가 배워야 할 것들을 대신 해버린다.
>
> 이 시리즈는 Spring을 오래 써온 개발자들이 **자기도 모르는 사이 놓쳐버린 것들**을 되짚는 기록이다.

---

## 완성된 문서

| # | 문서 | 주제 |
|---|------|------|
| 1 | [java-vs-nodejs-thread-model.md](../java/java-vs-nodejs-thread-model.md) | Java 멀티스레드 vs Node.js 이벤트루프 — 구조부터 비교 |
| 2 | [spring-mvc-thread-safety.md](./spring-mvc-thread-safety.md) | Spring MVC와 멀티스레드 — Tomcat, 스레드 안전성, @Async |
| 3 | [spring-hidden-internals.md](./spring-hidden-internals.md) | Spring이 숨긴 내부 동작 — @Transactional, @Autowired, HikariCP |
| 4 | [spring-design-patterns.md](./spring-design-patterns.md) | Spring이 숨긴 디자인 패턴 — Singleton, Proxy, Observer 등 |
| 5 | [spring-webflux-event-loop.md](./spring-webflux-event-loop.md) | Spring WebFlux — Java도 이벤트루프를 쓴다 (Netty, Mono/Flux) |

---

## 향후 작성 예정 (목차만)

### 5. 네트워크/HTTP 기초 — RestTemplate이 숨긴 것들
- TCP 소켓 연결, HTTP 헤더/바디 직접 파싱
- Connection Pool (keep-alive) 동작 원리
- 다른 언어(Python, Go)에서는 직접 구현하는 것들
- `RestTemplate` → `WebClient` → `HttpClient` 직접 사용 비교

### 6. 보안 인증/인가 — Spring Security가 숨긴 것들
- JWT 토큰 구조 (Header.Payload.Signature) 직접 생성/검증
- Session 관리, CSRF 토큰 원리
- Filter Chain 직접 구현해보기
- `@PreAuthorize`, `SecurityContextHolder` 내부 동작

### 7. 직렬화/역직렬화 — Jackson이 숨긴 것들
- Reflection으로 필드 탐색하는 원리
- Custom Serializer/Deserializer 직접 작성
- 성능 이슈 (Reflection 비용, DTO 설계 실수)
- `@JsonProperty`, `@JsonIgnore` 내부에서 일어나는 일

### 8. 예외 처리 전략 — @ControllerAdvice가 숨긴 것들
- 예외 전파 (Exception Propagation) 스택 읽기
- Checked vs Unchecked Exception 구분 실전
- 직접 Filter/Interceptor에서 예외 처리하기
- `@ExceptionHandler` 없이 예외를 어떻게 다루는가

### 9. 빌드/의존성 관리 — Maven/Gradle이 숨긴 것들
- ClassPath란 무엇인가, JVM이 클래스를 찾는 방법
- JAR Hell, 버전 충돌, 전이 의존성(transitive dependency)
- `mvn dependency:tree`로 숨겨진 의존성 보기
- 직접 javac + jar 명령어로 빌드해보기

---

## 시리즈 철학

Spring 없이 직접 구현해보는 경험이 없으면, Spring이 무엇을 해주는지 영원히 모른다.

```
Spring을 잘 쓰려면 → Spring이 뭘 하는지 알아야 한다
Spring이 뭘 하는지 알려면 → Spring 없이 해봐야 한다
```

각 문서는 이 구조를 따른다:
1. Spring에서 어떻게 쓰는가 (익숙한 것)
2. 내부에서 실제로 무슨 일이 일어나는가
3. 다른 언어/프레임워크에서는 어떻게 하는가
4. Spring 없이 직접 구현한다면
