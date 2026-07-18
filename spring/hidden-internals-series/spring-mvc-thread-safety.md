# Spring 개발자인데 멀티스레드를 몰랐다 (2편) — Spring MVC 요청은 어떤 스레드 위에서 실행되는가

> 1편에서 Java 멀티스레드와 동기화 메커니즘을 다뤘다. 2편에서는 Spring MVC가 내부적으로 어떻게 멀티스레드로 동작하는지, 그리고 개발자가 놓치기 쉬운 스레드 안전성 문제를 짚는다.

---

## 들어가며 — 우리는 이미 멀티스레드를 쓰고 있었다

Spring을 처음 배울 때 `@Controller`, `@Service`, `@Repository`를 배우고, HTTP 요청을 받아 DB에서 데이터를 꺼내 응답하는 흐름을 익힌다. 그런데 정작 **그 요청이 어떤 스레드 위에서 실행되는지** 설명하라고 하면 막히는 경우가 많다.

사실 Spring MVC는 처음부터 **멀티스레드 기반**이다. 특별히 뭔가를 설정하지 않아도, Tomcat이 요청마다 스레드를 꺼내 DispatcherServlet에 넘기는 방식으로 동시 요청을 처리한다. 기본값이 **200개 스레드**다.

이게 왜 중요한지, 직접 경험한 사례가 있다.

> **실전 경험**: Spring Framework를 사용하던 시절, `server.xml`에서 Tomcat 스레드 수를 너무 낮게 설정한 적이 있었다. 서비스가 외부 API를 호출하는 구간에서 간헐적으로 실패가 발생했고, 원인을 한참 찾다가 결국 스레드가 부족해서 요청이 대기 큐에서 타임아웃되고 있었다는 걸 알게 됐다. 스레드 설정을 정상으로 되돌리자 외부 API 호출 실패가 사라졌다.
>
> **멀티스레드를 이해하지 못한 채 설정을 건드리다 장애를 만든 경험이다.** 이 글은 그 부끄러움에서 시작됐다.

---

## 1. Tomcat ThreadPool — Spring MVC는 이미 멀티스레드

### 동작 방식

```
요청 A 들어옴 → Tomcat이 Thread-1 꺼내서 → DispatcherServlet 실행
요청 B 들어옴 → Tomcat이 Thread-2 꺼내서 → DispatcherServlet 실행
                                    ↓
                         둘 다 같은 OrderService Bean 공유 (싱글톤)
                         → 인스턴스 변수에 상태 저장하면 위험!
```

- 기본 스레드 수: **200개** (server.tomcat.threads.max)
- 스레드 부족 시 요청은 **대기 큐**에서 대기
- DispatcherServlet → Controller → Service → Repository 전체 흐름이 **하나의 스레드 위에서 실행**

### ThreadPool이 필요한 이유

```
스레드를 매번 new Thread()로 생성하면?
  → Thread 생성/소멸 비용: 약 수십 ~ 수백 μs
  → 1000 req/s 서버라면 스레드 생성만으로 상당한 CPU 낭비

ThreadPool을 쓰면?
  → 미리 만들어둔 스레드 재사용
  → 생성 비용 제거, 스레드 수 제한으로 안정성 확보
```

```
[ThreadPool 동작]

요청 → [대기 큐(Queue)] → [Thread-1] → 처리 완료 → 다시 풀로 반환
                        → [Thread-2] → 처리 완료 → 다시 풀로 반환
                        → [Thread-N] → ...

큐가 가득 찼을 때 새 요청 → RejectedExecutionException (or 타임아웃)
```

---

## 2. 스레드 안전성 — Spring Bean에서 놓치기 쉬운 것들

### 스레드 안전성 체크리스트

| 항목 | 안전 여부 | 이유 |
|---|---|---|
| `@Service` 내 지역 변수 | ✅ 안전 | 스레드 Stack에 독립 |
| `@Service` 내 인스턴스 변수 | ❌ 위험 | 싱글톤 → Heap 공유 |
| `ThreadLocal` 사용 | ✅ 안전 | 스레드별 독립 저장소 |
| `static` 변수 | ❌ 위험 | 전체 공유 |
| `synchronized` / `AtomicInteger` | ✅ 안전 | 동기화 처리됨 |

### ThreadLocal 이란

스레드마다 독립적인 저장 공간을 제공하는 도구다.

```java
// Spring Security 내부 구현 방식
public class SecurityContextHolder {
    // 스레드별로 완전히 독립된 값 저장
    private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

    public static void setContext(SecurityContext context) {
        contextHolder.set(context);
    }

    public static SecurityContext getContext() {
        return contextHolder.get();
    }

    // 요청 처리 완료 후 반드시 clear! (스레드 재사용이므로 안 지우면 이전 유저 정보 남음)
    public static void clearContext() {
        contextHolder.remove();
    }
}
```

> **왜 ThreadLocal?** 요청 A의 유저 정보와 요청 B의 유저 정보가 같은 Bean에 저장되면 혼재된다. ThreadLocal은 같은 Bean이라도 스레드마다 다른 값을 보장한다.

---

## 3. @Async — Spring 환경에서 가장 간단한 비동기 처리

```java
@SpringBootApplication
@EnableAsync  // 활성화 필수
public class App { ... }

@Service
public class EmailService {

    @Async  // 이것만 붙이면 별도 스레드에서 실행
    public CompletableFuture<Void> sendEmail(String to) {
        mailSender.send(to);
        return CompletableFuture.completedFuture(null);
    }
}
```

```yaml
# application.yml
spring:
  task:
    execution:
      pool:
        core-size: 8
        max-size: 32
        queue-capacity: 100
```

### @Async 주의사항

```java
// ❌ 같은 클래스 내부 호출은 @Async 동작 안 함 (프록시 우회)
@Service
public class MyService {
    public void outer() {
        this.inner();  // AOP 프록시를 거치지 않아 @Async 무시됨
    }

    @Async
    public void inner() { ... }
}

// ✅ 별도 Bean으로 분리해야 함
@Service
public class MyService {
    @Autowired
    private AsyncWorker asyncWorker;  // 별도 Bean

    public void outer() {
        asyncWorker.inner();  // 프록시를 통해 호출 → @Async 동작
    }
}
```

---

## 4. Spring MVC vs Spring WebFlux

| 항목 | Spring MVC | Spring WebFlux |
|---|---|---|
| 스레드 모델 | 요청당 스레드 (Tomcat) | 이벤트루프 (Node.js 방식) |
| 프로그래밍 모델 | 동기/블로킹 | 비동기/논블로킹 |
| 코드 스타일 | 익숙하고 직관적 | Mono/Flux 러닝커브 있음 |
| 적합한 상황 | 일반 CRUD, 대부분 서비스 | 초고성능 I/O, 스트리밍 |

### WebFlux의 Mono / Flux란

```java
// Mono: 0 또는 1개의 데이터를 비동기로 처리
Mono<User> userMono = userRepository.findById(id);  // 논블로킹

// Flux: 0 ~ N개의 데이터 스트림
Flux<Order> orderFlux = orderRepository.findAll();

// 체이닝 (함수형)
Mono<String> result = userRepository.findById(id)
    .map(user -> user.getName())
    .switchIfEmpty(Mono.error(new UserNotFoundException()));
```

> WebFlux는 **"나 지금 결과를 구독할게, 준비되면 알려줘"** 방식이다. 스레드가 I/O를 기다리며 블로킹하지 않는다.

### 언제 WebFlux를 선택하는가

```
WebFlux 고려 대상:
  - 수만 건의 동시 연결 처리 (SSE, WebSocket, 채팅)
  - 마이크로서비스 간 비동기 호출이 많은 경우
  - 팀 전체가 리액티브 프로그래밍에 익숙한 경우

Spring MVC로 충분한 경우:
  - 일반적인 CRUD API
  - 팀이 리액티브에 익숙하지 않은 경우
  - 빠른 개발과 유지보수성이 우선인 경우
```

> **결론**: 대부분의 SI/일반 서비스는 Spring MVC로 충분. 팀 전체가 WebFlux에 익숙하지 않으면 오히려 유지보수 어려워짐.

---

## 5. 전체 흐름 요약

```
[클라이언트 요청]
        ↓
[Tomcat ThreadPool] — 스레드 할당
        ↓
[DispatcherServlet] — 단일 진입점
        ↓
[HandlerMapping] — URL → Controller 매핑
        ↓
[Controller] — 요청 파싱, Service 호출
        ↓
[Service] — 비즈니스 로직 (싱글톤 Bean)
    ├── @Async 필요 시 → 별도 스레드로 위임
    ├── CompletableFuture → 비동기 병렬 처리
    └── 일반 로직 → 현재 스레드에서 처리
        ↓
[Repository] — DB 접근 (JPA, MyBatis 등)
        ↓
[응답 반환] — JSON 직렬화 (REST) or View 렌더링 (MVC)
        ↓
[스레드 반환] — ThreadPool로 반납
```

**핵심 원칙:**
- Controller, Service, Repository는 **상태를 갖지 않는다** (stateless)
- 요청별 데이터는 **메서드 파라미터(Stack)** 또는 **ThreadLocal** 에 보관
- 공유 자원(카운터, 캐시 등)은 **동기화 처리** 필수

---

**← 1편: [Java는 왜 멀티스레드고 Node.js는 왜 싱글스레드인가](java-vs-nodejs-thread-model.md)**
