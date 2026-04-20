# Spring 개발자인데 스레드를 몰랐다 — Node.js, Java, Spring 비교

> Java/Spring으로 밥 먹고 살면서도 "요청이 어떤 스레드 위에서 실행되는가"를 제대로 설명하지 못했다. Node.js와 비교하며 스레드 모델을 처음부터 다시 정리했다.

---

## 1. Node.js — 싱글스레드 이벤트루프

Node.js는 스레드를 **1개만** 쓴다. 대신 I/O 작업은 OS에 위임하고 기다리지 않는다.

핵심 동작 방식은 이렇다. 파일을 읽는 코드가 있을 때:

```js
fs.readFile('a.txt', (err, data) => {
    console.log(data);         // 3. 나중에 실행
});
console.log('다음 코드 실행'); // 2. 먼저 출력됨
                               // 1. readFile은 OS에 맡기고 즉시 다음 줄로 넘어감
```

파일을 다 읽을 때까지 기다리지 않고 다음 코드를 실행한다. 파일 읽기가 완료되면 그때 콜백이 실행된다. 이 구조 덕분에 스레드 1개로 수천 개의 I/O 요청을 동시에 처리할 수 있다.

- **강점**: 스레드 없이 수천 개 동시 요청 처리 가능
- **약점**: 이미지 리사이즈, 암호화 같은 CPU 집약 작업이 끼어들면 **그동안 다른 요청을 처리하지 못함**

---

## 2. Java — 멀티스레드

Java는 CPU 코어를 최대한 활용하기 위해 **여러 스레드**를 운영한다. 각 스레드는 OS 스케줄러에 의해 서로 다른 코어에서 진짜 병렬로 실행된다.

```
JVM Process
├── Heap (공유) ← 모든 스레드가 접근 → 동기화 필요
├── Thread-1 (독립 Stack)
├── Thread-2 (독립 Stack)
└── Thread-3 (독립 Stack)
```

Heap은 공유되기 때문에 동시 접근 시 Race Condition이 발생한다.

```java
class Counter {
    int count = 0; // Heap 공유

    void increment() {
        count++; // 사실 3단계 연산 (읽기 → +1 → 쓰기)
                 // 두 스레드가 동시에 읽으면 둘 다 같은 값으로 +1
    }
}
```

이를 막기 위한 동기화 도구들:

| 도구 | 용도 |
|---|---|
| `synchronized` | 가장 기본적인 락, 한 번에 하나의 스레드만 진입 |
| `volatile` | 캐시 무시하고 메인 메모리 직접 읽기/쓰기 (가시성 보장) |
| `ReentrantLock` | 타임아웃, 공정성 설정 등 고급 제어 |
| `AtomicInteger` | CPU 레벨 CAS 연산, 락 없이 원자성 보장 |

### 실무에서 멀티스레드 쓰는 방법

```java
// 1. ExecutorService — 스레드풀 기반 (실무 기본)
ExecutorService executor = Executors.newFixedThreadPool(4);
Future<String> future = executor.submit(() -> "결과값");
String result = future.get(); // 블로킹 대기

// 2. CompletableFuture — 현재 권장 (Java 8+)
CompletableFuture.supplyAsync(() -> userService.findById(1L))
    .thenApply(user -> user.getName())
    .thenAccept(name -> System.out.println("이름: " + name));

// 여러 API 동시 호출 후 합산
CompletableFuture<User> userFuture  = CompletableFuture.supplyAsync(() -> userService.get(id));
CompletableFuture<List<Order>> orderFuture = CompletableFuture.supplyAsync(() -> orderService.getList(id));

CompletableFuture.allOf(userFuture, orderFuture).thenRun(() -> {
    User user          = userFuture.join();
    List<Order> orders = orderFuture.join();
});
```

---

## 3. Spring MVC — 우리는 이미 멀티스레드를 쓰고 있었다

Spring MVC를 쓴다면 멀티스레드는 이미 동작 중이다. 아무것도 설정하지 않아도, **Tomcat이 요청마다 스레드를 꺼내 처리**한다.

```
요청 A 들어옴 → Tomcat이 Thread-1 꺼내서 → DispatcherServlet 실행
요청 B 들어옴 → Tomcat이 Thread-2 꺼내서 → DispatcherServlet 실행
                                    ↓
                         둘 다 같은 OrderService Bean 공유 (싱글톤)
```

- 기본 스레드 수: **200개** (`server.tomcat.threads.max`)
- Controller → Service → Repository 전체 흐름이 **하나의 스레드 위에서 실행**
- 스레드 처리 완료 후 ThreadPool로 **반납**되어 재사용

### 이게 왜 중요한가

> 실전 경험: `server.xml`에서 Tomcat 스레드 수를 낮게 설정했다가 외부 API 호출 구간에서 간헐적 실패가 발생했다. 원인은 스레드 부족으로 요청이 대기 큐에서 타임아웃된 것이었다. 스레드 모델을 이해하지 못한 채 설정을 건드리다 장애를 만든 경험이다.

Bean에서 놓치기 쉬운 스레드 안전성:

| 항목 | 안전 여부 | 이유 |
|---|---|---|
| `@Service` 내 지역 변수 | ✅ 안전 | 스레드 Stack에 독립 |
| `@Service` 내 인스턴스 변수 | ❌ 위험 | 싱글톤 → Heap 공유 |
| `ThreadLocal` | ✅ 안전 | 스레드별 독립 저장소 |
| `static` 변수 | ❌ 위험 | 전체 공유 |

Spring Security의 `SecurityContextHolder`가 `ThreadLocal`을 쓰는 이유가 이것이다. 요청마다 다른 유저 정보를 같은 Bean에서 격리해야 하기 때문이다.

### @Async — 백그라운드 작업

```java
@SpringBootApplication
@EnableAsync
public class App { ... }

@Service
public class EmailService {
    @Async // 이것만 붙이면 별도 스레드에서 실행
    public CompletableFuture<Void> sendEmail(String to) {
        mailSender.send(to);
        return CompletableFuture.completedFuture(null);
    }
}
```

> 주의: 같은 클래스 내부 호출은 `@Async`가 동작하지 않는다. Spring AOP 프록시를 우회하기 때문이다. 반드시 별도 Bean으로 분리해야 한다.

---

## 4. Spring WebFlux — Java도 이벤트루프를 쓴다

Spring MVC만 쓰다 보면 "Java = 멀티스레드"로 굳어버린다. 그런데 **Spring WebFlux**는 Node.js처럼 이벤트루프로 동작한다.

```
Spring MVC:  요청 → Tomcat 스레드 할당 → 처리 → 스레드 반납
Spring WebFlux: 요청 → Netty 이벤트루프 → 논블로킹 처리
```

기반 구조:

```
Spring WebFlux
    ↓
Project Reactor (Mono / Flux)
    ↓
Netty (이벤트루프 기반 NIO 서버)
    ↓
Java NIO (JDK 논블로킹 I/O)
```

컨트롤러가 값을 바로 반환하지 않고 **"나중에 값이 올 것"** 이라는 약속(`Mono`/`Flux`)을 반환한다:

```java
// Spring MVC — 스레드가 DB 응답까지 대기
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id);
}

// Spring WebFlux — 즉시 Mono 반환, 스레드 해방
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return userRepository.findById(id);
}
```

| 항목 | Spring MVC | Spring WebFlux |
|---|---|---|
| 서버 | Tomcat | Netty |
| 스레드 모델 | 요청당 1스레드 | 이벤트루프 (소수 스레드) |
| I/O | 블로킹 | 논블로킹 |
| 반환 타입 | `String`, `ResponseEntity` | `Mono<T>`, `Flux<T>` |
| 적합한 상황 | 일반 CRUD | 대규모 동시 연결, 스트리밍 |

---

## 정리 — 세 가지 스레드 모델

| | Node.js | Java (Spring MVC) | Java (Spring WebFlux) |
|---|---|---|---|
| 스레드 수 | 1개 | 요청당 1개 (기본 200개 풀) | 소수 (코어 수 × 2) |
| I/O 처리 | libuv, 논블로킹 | 블로킹 대기 | Netty NIO, 논블로킹 |
| CPU 작업 | 서버 멈춤 위험 | 멀티코어 활용 | 별도 스케줄러 필요 |
| 코드 스타일 | 콜백 / async-await | 동기, 직관적 | Mono/Flux 체이닝 |
| 주요 사용처 | 실시간, API Gateway | 일반 CRUD | 대규모 동시 연결 |

**어느 것이 더 좋은가?** 정답 없다. 대부분의 서비스는 Spring MVC로 충분하다. 수만 건 동시 연결이나 스트리밍이 필요할 때 WebFlux를 고려한다. WebFlux는 팀 전체가 리액티브 패턴에 익숙하지 않으면 오히려 유지보수가 어려워진다.
