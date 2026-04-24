# Node.js 싱글스레드에서 Spring 멀티스레드까지 — 멀티스레드를 제대로 이해하기

> Node.js의 이벤트루프와 Java의 멀티스레드. 겉보기엔 완전히 다르지만, 둘 다 동시성을 처리하는 방식일 뿐이다.
> 이 글에서는 두 플랫폼의 스레드 모델을 비교하면서, 개발자가 자주 놓치는 Spring MVC의 멀티스레드 안전성 문제까지 다룬다.

---

## 0. 들어가며 — 우리는 이미 멀티스레드를 쓰고 있었다

Spring을 처음 배울 때 `@Controller`, `@Service`, `@Repository` 를 배우고, HTTP 요청을 받아 DB에서 데이터를 꺼내 응답하는 흐름을 익힌다. 그런데 정작 **그 요청이 어떤 스레드 위에서 실행되는지** 설명하라고 하면 막히는 경우가 많다.

사실 Spring MVC는 처음부터 **멀티스레드 기반**이다. 특별히 뭔가를 설정하지 않아도, Tomcat이 요청마다 스레드를 꺼내 DispatcherServlet에 넘기는 방식으로 동시 요청을 처리한다. 기본값이 **200개 스레드**다.

> **실전 경험**: Spring Framework를 사용하던 시절, `server.xml`에서 Tomcat 스레드 수를 너무 낮게 설정한 적이 있었다. 서비스가 외부 API를 호출하는 구간에서 간헐적으로 요청이 실패했고, 원인을 한참 찾다가 결국 스레드가 부족해서 요청이 대기 큐에서 타임아웃되고 있었다는 걸 알게 됐다. 스레드 설정을 정상으로 되돌리자 외부 API 호출 실패가 사라졌다.
>
> **멀티스레드를 이해하지 못한 채 설정을 건드리다 장애를 만든 경험이다.** 이것이 이 글의 시작점이다.

---

## 1. 왜 스레드 모델이 중요한가?

두 가지 극단적인 설계 방식이 있다:

- **Node.js**: 스레드 1개, 멀티플렉싱(이벤트루프)으로 높은 동시성 처리
- **Java**: 요청당 스레드 1개, 각 스레드가 블로킹 I/O 처리

어느 쪽이 더 좋을까? **정답은 없다.** 워크로드 특성에 따라 다르다.

| 항목 | Node.js 싱글스레드 | Java 멀티스레드 |
|---|---|---|
| **동시 요청 처리** | 이벤트루프 → Callback Queue | ThreadPool → 스레드 할당 |
| **I/O 집약적 작업** | 매우 유리 | 스레드 생성 비용 있음 |
| **CPU 집약적 작업** | 전체 서버 블로킹 위험 | 멀티코어 활용 유리 |
| **공유 자원 관리** | 기본적으로 불필요 | 동기화 처리 필수 |
| **개발 난이도** | 콜백 지옥, 비동기 추적 어려움 | 직관적 동기 코드 |

---

## 2. Node.js: 싱글스레드 이벤트루프

### 왜 싱글스레드인가?

브라우저 환경에서 출발한 JavaScript는 DOM 조작 등 멀티스레드가 오히려 복잡성을 높이는 환경에서 설계됐다. 서버(Node.js)에서도 이 철학을 이어받아 **"스레드를 늘리는 대신 기다리는 시간을 없애자"** 는 방향을 선택했다.

> **비유**: 음식점 주방에서 요리사가 1명인데, 물이 끓기를 기다리면서 다른 재료를 손질한다. 멍하니 냄비만 바라보지 않는다.

### 동작 원리

```
HTTP 요청 → Call Stack(JS 실행) → I/O 만나면 libuv에 위임 → 즉시 다음 코드 실행
                                                   ↓
                                         작업 완료 → Callback Queue
                                                   ↓
                                         Call Stack 비면 → Event Loop가 꺼내 실행
```

### 핵심 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **Call Stack** | JS 코드 실행 공간. 싱글스레드, 한 번에 하나 |
| **Event Loop** | Call Stack이 비면 Callback Queue에서 꺼내 실행. 계속 반복 |
| **Callback Queue** | 완료된 비동기 작업의 콜백 대기열 (FIFO) |
| **libuv / OS** | 파일 I/O, 네트워크, 타이머 등 비동기 처리 위임 |

### 코드 예시

```js
// 블로킹 없이 흘러감
fs.readFile('a.txt', (err, data) => {
  console.log(data);  // 나중에 실행
});
console.log('다음 코드 즉시 실행');  // 먼저 출력됨
```

실행 순서:
1. `readFile()` 호출 → libuv에 위임 (논블로킹)
2. `console.log('다음...')` 즉시 실행
3. 파일 I/O 완료 후 콜백 실행

### 강점 / 약점

**강점:**
- I/O가 많은 서버에서 스레드 생성 없이 수천 개 요청 처리 가능
- 메모리 효율적 (스레드 오버헤드 없음)
- 개발이 간단함 (기본적으로 동기화 고민 없음)

**약점:**
- CPU 집약 작업(이미지 리사이즈, 암호화)이 Call Stack을 점유하면 **전체 서버가 멈춤**
- 콜백 체인이 깊어지면 추적 어려움
- 비동기 에러 처리가 복잡함

---

## 3. Java: 멀티스레드 기반 병렬 처리

### 왜 멀티스레드인가?

CPU 코어가 여러 개인 현대 하드웨어를 최대한 활용하기 위해서다. 각 스레드는 OS 스케줄러에 의해 서로 다른 코어에서 **진짜 병렬**로 실행될 수 있다.

> **비유**: 주방에 요리사가 여러 명이다. 각자 다른 요리를 동시에 한다. 단, 냉장고(공유 자원)를 같이 쓰므로 충돌 방지 규칙이 필요하다.

### JVM 메모리 구조

```
JVM Process
├── Heap (공유) ← 모든 스레드가 접근 → 동기화 필수
│   ├── 객체 인스턴스
│   └── static 변수
├── Thread-1 (독립 Stack)  ← 지역 변수, 메서드 호출 프레임
├── Thread-2 (독립 Stack)
└── Thread-3 (독립 Stack)
```

**핵심 포인트:**
- 각 스레드는 독립적인 **Stack, PC Register** 보유
- **Heap만 공유** → 동기화 필요

### 스레드 생명주기

```
NEW → start() → RUNNABLE ←→ BLOCKED/WAITING/TIMED_WAITING
                    ↓
                TERMINATED
```

| 상태 | 설명 |
|---|---|
| **NEW** | `new Thread()` 생성만 된 상태 |
| **RUNNABLE** | 실행 중 또는 OS 스케줄링 대기 중 |
| **BLOCKED** | `synchronized` 락 획득 대기 |
| **WAITING** | `wait()`, `join()` 등으로 대기 |
| **TIMED_WAITING** | `sleep(ms)`, `wait(ms)` 등 시간 제한 대기 |
| **TERMINATED** | 실행 완료 또는 예외 종료 |

---

## 4. Spring MVC: 요청당 스레드 모델

이제 Node.js와 Java를 넘어 **Spring MVC가 실제로 멀티스레드를 어떻게 활용하는지** 살펴보자.

### Tomcat ThreadPool 동작 방식

```
요청 A 들어옴 → Tomcat이 Thread-1 꺼내서 → DispatcherServlet 실행
요청 B 들어옴 → Tomcat이 Thread-2 꺼내서 → DispatcherServlet 실행
                                    ↓
                         둘 다 같은 OrderService Bean 공유 (싱글톤)
                         → 인스턴스 변수에 상태 저장하면 위험!
```

**주요 특징:**
- 기본 스레드 수: **200개** (`server.tomcat.threads.max`)
- 스레드 부족 시 요청은 **대기 큐**에서 대기
- DispatcherServlet → Controller → Service → Repository 전체 흐름이 **하나의 스레드 위에서 실행**

### 왜 ThreadPool이 필요한가?

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

요청 → [대기 큐] → [Thread-1] → 처리 완료 → 다시 풀로 반환
                  → [Thread-2] → 처리 완료 → 다시 풀로 반환
                  → [Thread-N] → ...

큐가 가득 찼을 때 새 요청 → RejectedExecutionException (타임아웃)
```

### 실제 요청 흐름

```
[클라이언트 요청]
        ↓
[Tomcat ThreadPool] — 스레드 할당 (부족 시 대기 큐)
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

---

## 5. Spring MVC에서 놓치기 쉬운 스레드 안전성

Spring MVC는 멀티스레드 환경이다. 즉, **여러 요청이 동시에 같은 Bean의 메서드를 실행할 수 있다.**

### 스레드 안전성 체크리스트

| 항목 | 안전 여부 | 이유 |
|---|---|---|
| `@Service` 내 **지역 변수** | ✅ 안전 | 스레드 Stack에 독립 |
| `@Service` 내 **인스턴스 변수** | ❌ 위험 | 싱글톤 → Heap 공유 |
| **ThreadLocal** | ✅ 안전 | 스레드별 독립 저장소 |
| **static 변수** | ❌ 위험 | 전체 공유 |
| **synchronized / AtomicInteger** | ✅ 안전 | 동기화 처리됨 |

### 위험한 패턴: 인스턴스 변수 사용

```java
@Service
public class OrderService {
    // ❌ 위험: 모든 스레드가 같은 인스턴스 변수 공유
    private List<Order> tempOrders = new ArrayList<>();

    public void processOrder(Order order) {
        tempOrders.add(order);  // 요청 A와 B가 동시에 실행되면?
        // ...
        tempOrders.clear();
        // → 요청 B의 데이터가 요청 A에 의해 삭제될 수 있음 (Race Condition)
    }
}
```

**문제점:**
- Thread-1이 `tempOrders.add()`를 실행 중일 때
- Thread-2도 같은 `tempOrders`에 접근
- 데이터 무결성 보장 불가

### 안전한 패턴: 지역 변수 사용

```java
@Service
public class OrderService {
    public void processOrder(Order order) {
        // ✅ 안전: 메서드 호출할 때마다 새로운 지역 변수 생성
        // → 각 스레드의 Stack에 독립적으로 저장
        List<Order> tempOrders = new ArrayList<>();
        tempOrders.add(order);
        // ...
        tempOrders.clear();
        // → 스레드 간 간섭 없음
    }
}
```

**왜 안전한가:**
- 메서드가 호출될 때마다 **새로운 지역 변수 생성**
- 각 스레드는 자신의 **Stack에만 접근**
- 다른 스레드의 Stack은 절대 건드릴 수 없음

### Race Condition 심화: 원자성 문제

```java
class Counter {
    int count = 0;  // Heap에 저장 (공유)

    void increment() {
        count++;  // 이건 사실 3단계 연산
        // 1. count 읽기 (CPU 캐시 또는 메인 메모리에서)
        // 2. +1 계산
        // 3. count에 쓰기
    }
}
```

**시나리오:**
```
Thread-1: count = 5 읽음
Thread-2: count = 5 읽음           ← 같은 값을 동시에 읽음!
Thread-1: 5 + 1 = 6 계산, 쓰기 (count = 6)
Thread-2: 5 + 1 = 6 계산, 쓰기 (count = 6)

예상 결과: 7 (두 번 증가)
실제 결과: 6 (한 번만 증가한 것처럼 보임) ← Race Condition!
```

---

## 6. Java 동기화 메커니즘

### synchronized — 가장 기본적인 락

```java
class Counter {
    int count = 0;

    // 이 메서드는 한 번에 하나의 스레드만 실행 가능
    synchronized void increment() {
        count++;
    }
}
```

**동작 방식:**
1. 스레드가 메서드 진입 → **모니터 락(Monitor Lock)** 획득
2. 임계 영역(critical section) 실행
3. 메서드 종료 → 락 반납
4. 다른 스레드는 락이 반납될 때까지 **BLOCKED** 상태

**세밀한 제어: 블록 단위 락**

```java
void doSomething() {
    // 여기는 여러 스레드 동시 실행 가능
    prepare();

    synchronized (this) {
        // 이 블록만 한 번에 하나의 스레드가 실행
        count++;
    }

    // 여기도 동시 실행 가능
    after();
}
```

### volatile — 가시성 문제 해결

CPU 캐시로 인한 문제가 있다:

```
CPU Core 1          CPU Core 2
    ↓                   ↓
L1 Cache            L1 Cache    ← 각 코어가 캐시를 따로 가짐
    ↓                   ↓
           Heap (RAM)
```

Thread-1이 값을 변경해도 Thread-2가 오래된 캐시 값을 읽을 수 있다.

```java
// volatile: 캐시 사용 안 하고 항상 메인 메모리(Heap) 직접 읽기/쓰기
volatile boolean running = true;

// Thread-1
running = false;

// Thread-2
while (running) {  // volatile 없으면 변경 감지 못할 수 있음
    doWork();
}
```

> **주의**: `volatile`은 가시성만 보장. `count++` 같은 복합 연산은 여전히 Race Condition 발생.

### ReentrantLock — 유연한 락

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();  // 반드시 finally에서 해제
        }
    }

    // tryLock: 락 못 얻으면 기다리지 않고 false 반환
    void tryIncrement() {
        if (lock.tryLock()) {
            try {
                count++;
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println("락 획득 실패, 다음에 재시도");
        }
    }
}
```

| 항목 | `synchronized` | `ReentrantLock` |
|---|---|---|
| 사용 난이도 | 쉬움 | 복잡함 |
| 타임아웃 | 불가 | `tryLock(timeout)` 가능 |
| 공정성 설정 | 불가 | `new ReentrantLock(true)` |
| 인터럽트 가능 | 불가 | `lockInterruptibly()` |

### AtomicInteger — 락 없는 원자 연산

```java
import java.util.concurrent.atomic.AtomicInteger;

// synchronized 없이도 스레드 안전한 카운터
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();   // ++count (원자적)
count.getAndIncrement();   // count++ (원자적)
count.addAndGet(5);        // count += 5 (원자적)

// CAS (Compare-And-Swap): 기대값과 같을 때만 변경
count.compareAndSet(10, 20);  // count가 10이면 20으로 변경
```

> CPU 레벨의 **CAS 명령어**를 사용해서 락 없이도 원자성을 보장한다. `synchronized`보다 성능이 좋다.

---

## 7. ThreadLocal — Spring Security의 비결

스레드마다 독립적인 저장 공간을 제공하는 도구다. Spring Security가 사용자 정보를 스레드마다 다르게 유지하는 원리가 바로 이것이다.

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

**실제 사용 예시:**

```java
@RestController
public class MyController {

    @GetMapping("/api/profile")
    public UserProfile getProfile() {
        // ThreadLocal에서 현재 요청 스레드의 사용자만 가져옴
        SecurityContext context = SecurityContextHolder.getContext();
        Authentication auth = context.getAuthentication();
        User currentUser = (User) auth.getPrincipal();

        // Thread-1의 요청이면 유저 A, Thread-2의 요청이면 유저 B 조회
        return userService.getProfile(currentUser.getId());
    }
}
```

**왜 ThreadLocal?**

요청 A의 유저 정보와 요청 B의 유저 정보가 같은 Bean에 저장되면 혼재된다. ThreadLocal은 같은 Bean이라도 스레드마다 다른 값을 보장한다.

> **스레드 풀 사용 시 주의**: Tomcat은 스레드를 재사용하므로, 요청 처리 완료 후 **반드시 `clear()` 호출**해야 다음 요청에서 이전 데이터가 남지 않는다.

---

## 8. Spring MVC에서 비동기 처리하기

### @Async — 간단한 백그라운드 작업

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
        core-size: 8        # 항상 유지할 스레드 수
        max-size: 32        # 최대 스레드 수
        queue-capacity: 100 # 대기 큐 크기
```

**사용:**

```java
@PostMapping("/orders")
public ResponseEntity<?> createOrder(@RequestBody OrderRequest req) {
    Order order = orderService.save(req);

    // 이메일 발송은 별도 스레드에서 (응답 차단 없음)
    emailService.sendEmail(order.getEmail());

    return ResponseEntity.ok(order);
}
```

### @Async 주의사항: 프록시 우회

```java
// ❌ 같은 클래스 내부 호출은 @Async 동작 안 함 (프록시 우회)
@Service
public class MyService {
    public void outer() {
        this.inner();  // AOP 프록시를 거치지 않아 @Async 무시됨 → 동기 실행!
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

@Service
class AsyncWorker {
    @Async
    public void inner() { ... }
}
```

### CompletableFuture — 더 유연한 비동기

```java
// 여러 API 동시 호출 후 합산 (실무 패턴)
public UserDashboard getUserDashboard(Long userId) {
    CompletableFuture<User> userFuture =
        CompletableFuture.supplyAsync(() -> userService.getUser(userId));

    CompletableFuture<List<Order>> orderFuture =
        CompletableFuture.supplyAsync(() -> orderService.getOrders(userId));

    CompletableFuture<List<Review>> reviewFuture =
        CompletableFuture.supplyAsync(() -> reviewService.getReviews(userId));

    // 셋 다 완료된 시점에 결합
    return CompletableFuture.allOf(userFuture, orderFuture, reviewFuture)
        .thenApply(v -> {
            User user = userFuture.join();
            List<Order> orders = orderFuture.join();
            List<Review> reviews = reviewFuture.join();
            return new UserDashboard(user, orders, reviews);
        })
        .join();  // 모든 작업 완료 대기
}
```

**주요 메서드:**

| 메서드 | 설명 |
|---|---|
| `supplyAsync(Supplier)` | 비동기 실행, 반환값 있음 |
| `runAsync(Runnable)` | 비동기 실행, 반환값 없음 |
| `thenApply(Function)` | 결과 변환 |
| `thenCompose(Function)` | 결과로 새 CompletableFuture 생성 (flatMap) |
| `thenAccept(Consumer)` | 결과 소비, 반환 없음 |
| `allOf(futures...)` | 전부 완료 대기 |
| `anyOf(futures...)` | 하나라도 완료되면 |
| `exceptionally(Function)` | 예외 처리 |
| `join()` | 블로킹 대기 |

---

## 9. Java 멀티스레드 사용 방법 (진화 순서)

### 1단계: Thread 직접 생성 (원시적, 거의 안 씀)

```java
Thread t = new Thread(() -> System.out.println("실행: " + Thread.currentThread().getName()));
t.start();
```

**문제:**
- 요청마다 스레드 새로 생성 → 생성/소멸 비용 큼
- 스레드 수 제어 불가 → 메모리 부족 가능
- 실무에서 거의 미사용

---

### 2단계: ExecutorService — ThreadPool (기본)

```java
// 고정 스레드 4개짜리 풀
ExecutorService executor = Executors.newFixedThreadPool(4);

// Runnable - 반환값 없음
executor.submit(() -> System.out.println("백그라운드 작업"));

// Callable - 반환값 있음
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "결과값";
});

String result = future.get(); // 블로킹 대기
executor.shutdown();
```

**Executors 팩토리 종류:**

| 팩토리 | 설명 | 주의 |
|---|---|---|
| `newFixedThreadPool(n)` | 고정 스레드 n개 | 큐 무제한 → OOM 가능 |
| `newCachedThreadPool()` | 필요할 때 생성, 60초 후 소멸 | 스레드 수 무제한 → 과부하 위험 |
| `newSingleThreadExecutor()` | 스레드 1개, 순서 보장 | 처리량 제한 |
| `newScheduledThreadPool(n)` | 스케줄링 (delay, 주기 실행) | — |

> 실무에서는 `ThreadPoolExecutor`를 직접 설정해 큐 크기를 명시적으로 제한하는 것을 권장한다 (Effective Java).

---

### 3단계: CompletableFuture — 현재 권장 (Java 8+)

위의 섹션 8 참고.

---

### 4단계: parallelStream — 간단한 CPU 작업

```java
int sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
```

**언제 유리한가:**
- 리스트가 충분히 클 때 (수천 개 이상)
- 각 원소 처리가 CPU 집약적일 때
- 순서가 중요하지 않을 때

**언제 오히려 느릴 수 있는가:**
- 리스트가 작을 때 (스레드 분배 오버헤드가 더 큼)
- I/O 작업 포함 시 (ForkJoinPool이 I/O에 최적화되지 않음)
- 순서 보장이 필요할 때

---

## 10. Spring MVC vs Spring WebFlux

이제 **Node.js의 이벤트루프 방식을 Java에서도 쓸 수 있는가?** 에 대한 답을 보자.

### Spring WebFlux: Java의 이벤트루프

Spring WebFlux는 **Node.js처럼 비동기/논블로킹 I/O를 사용**하는 Spring 프레임워크다. Netty 라이브러리를 기반으로 동작하며, 소수의 스레드로 많은 동시 연결을 처리한다.

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

### 비교 분석

| 항목 | Spring MVC | Spring WebFlux | Node.js |
|---|---|---|---|
| **스레드 모델** | 요청당 스레드 (Tomcat) | 소수 스레드 + 이벤트루프 | 1개 스레드 + 이벤트루프 |
| **프로그래밍 모델** | 동기/블로킹 | 비동기/논블로킹 | 비동기/논블로킹 |
| **코드 스타일** | 익숙하고 직관적 | Mono/Flux 러닝커브 있음 | 콜백/Promise 체인 |
| **I/O 집약 작업** | 스레드 수에 제한 | 적음 (소수 스레드로 처리) | 적음 (1개 스레드로 처리) |
| **CPU 집약 작업** | 멀티코어 활용 유리 | 별도 스케줄러 필요 | 전체 서버 블로킹 |
| **메모리** | 스레드당 ~1MB | 매우 적음 | 매우 적음 |

### 언제 WebFlux를 선택하는가

**WebFlux 고려 대상:**
- 수만 건의 동시 연결 처리 (SSE, WebSocket, 채팅)
- 마이크로서비스 간 비동기 호출이 많은 경우
- 팀 전체가 리액티브 프로그래밍에 익숙한 경우

**Spring MVC로 충분한 경우:**
- 일반적인 CRUD API (대부분의 엔터프라이즈 서비스)
- 팀이 리액티브에 익숙하지 않은 경우
- 빠른 개발과 유지보수성이 우선인 경우

> **결론**: 대부분의 SI/일반 서비스는 Spring MVC로 충분. 팀 전체가 WebFlux에 익숙하지 않으면 오히려 유지보수 어려워짐.

---

## 11. 상황별 선택 기준

| 상황 | 선택 |
|---|---|
| 일반적인 Spring REST API | Spring MVC (기본) |
| 여러 외부 API 동시 호출 후 합산 | `CompletableFuture.allOf` |
| 대용량 리스트 CPU 연산 | `parallelStream` |
| 이메일, 로그 기록 등 백그라운드 작업 | `@Async` |
| 수만 건 동시 연결 필요 | Spring WebFlux |
| `new Thread()` 직접 | 거의 쓸 일 없음 |

---

## 12. 실무 체크리스트

Spring MVC 서비스를 구축할 때 멀티스레드 안전성을 확보하기 위한 체크리스트:

- [ ] **ThreadPool 설정 확인**: Tomcat 스레드 수가 적절한가?
  ```yaml
  server:
    tomcat:
      threads:
        max: 200  # 워크로드에 맞게 조정
        min-spare: 10
  ```

- [ ] **Bean 설계**: @Service, @Repository가 상태를 갖지 않는가?
  ```java
  // ❌ 피해야 할 패턴
  @Service
  public class MyService {
      private List<Data> cache;  // 공유 상태
  }

  // ✅ 권장 패턴
  @Service
  public class MyService {
      private final CacheManager cacheManager;  // 스레드 안전한 구현
  }
  ```

- [ ] **공유 자원 보호**: 동기화가 필요한 부분은 처리했는가?
  ```java
  // 공유 상태 필요 시
  private final AtomicInteger counter = new AtomicInteger();
  // 또는
  private final Object lock = new Object();
  ```

- [ ] **@Async 사용**: 불필요한 블로킹이 없는가?
  ```java
  // 오래 걸리는 작업은 @Async로 분리
  @Async
  public void slowOperation() { ... }
  ```

- [ ] **ThreadLocal 정리**: Spring Security 등 ThreadLocal 사용 시 정리되는가?
  - Spring이 기본적으로 관리함 (Filter 레벨에서 clear())

- [ ] **외부 라이브러리**: DB 커넥션, HTTP 클라이언트 풀 설정은?
  ```yaml
  spring:
    datasource:
      hikari:
        maximum-pool-size: 10  # 동시 연결 수 제한
  ```

---

## 결론

| 플랫폼 | 전략 | 장점 | 단점 |
|---|---|---|---|
| **Node.js** | 1개 스레드 + 이벤트루프 | I/O 효율적, 메모리 적음 | CPU 작업 시 블로킹 위험 |
| **Java/Spring MVC** | 요청당 스레드 | 직관적 동기 코드, 멀티코어 활용 | 스레드 수 제한, 메모리 소비 |
| **Spring WebFlux** | Node.js 방식 | Java의 생태계 + 비동기 이점 | 학습 곡선, 팀 역량 필요 |

결국 **올바른 선택은 워크로드 특성에 따른 것**이다. 하지만 대부분의 엔터프라이즈 서비스는:

1. **Spring MVC**로 충분하고
2. **필요한 부분만 @Async로 최적화**하며
3. **Bean을 stateless로 설계**하고
4. **공유 자원은 동기화 처리**한다면

멀티스레드의 복잡성을 최소화하면서도 안전하고 높은 성능을 낼 수 있다.

이제 "Spring 개발자인데 멀티스레드를 몰랐다"는 말은 더 이상 하지 않기를 바란다. 🚀
