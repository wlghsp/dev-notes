# Spring 개발자인데 멀티스레드를 몰랐다 (1편) — Java는 왜 멀티스레드고 Node.js는 왜 싱글스레드인가

> Java/Spring으로 밥 먹고 살면서도 멀티스레드를 제대로 설명하지 못했다. 부끄러움을 인정하고 Node.js 이벤트루프와 비교하며 기초부터 동기화 메커니즘까지 다시 정리했다.

---

## 1. Node.js 싱글스레드 이벤트루프

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

### 구성 요소

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

### 강점 / 약점

- **강점**: I/O가 많은 서버에서 스레드 생성 없이 수천 개 요청 처리 가능
- **약점**: CPU 집약 작업(이미지 리사이즈, 암호화)이 Call Stack을 점유하면 **전체 서버가 멈춤**

---

## 2. Java 멀티스레드

### 왜 멀티스레드인가?

CPU 코어가 여러 개인 현대 하드웨어를 최대한 활용하기 위해서다. 각 스레드는 OS 스케줄러에 의해 서로 다른 코어에서 **진짜 병렬**로 실행될 수 있다.

> **비유**: 주방에 요리사가 여러 명이다. 각자 다른 요리를 동시에 한다. 단, 냉장고(공유 자원)를 같이 쓰므로 충돌 방지 규칙이 필요하다.

### JVM 메모리 구조

```
JVM Process
├── Heap (공유) ← 모든 스레드가 접근 → 동기화 필요
│   ├── 객체 인스턴스
│   └── static 변수
├── Thread-1 (독립 Stack)  ← 지역 변수, 메서드 호출 프레임
├── Thread-2 (독립 Stack)
└── Thread-3 (독립 Stack)
```

- 각 스레드는 독립적인 **Stack, PC Register** 보유
- **Heap만 공유** → `synchronized`, `Lock`, `volatile` 등 동기화 필요

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

## 3. Java vs Node.js 비교

| 항목 | Java 멀티스레드 | Spring WebFlux | Node.js 싱글스레드 |
|---|---|---|---|
| 스레드 수 | 여러 개 (진짜 병렬) | 소수 (이벤트루프) | 1개 |
| I/O 처리 | 스레드가 블로킹 대기 | Netty NIO, 논블로킹 | libuv에 위임, 논블로킹 |
| CPU 집약 작업 | 멀티코어 활용 유리 | 블로킹 위험 (별도 스케줄러 필요) | 전체 서버 블로킹 위험 |
| 동기화 필요 | 공유 자원에 필요 | 공유 상태 최소화 설계 | 기본적으로 불필요 |
| 메모리 | 스레드당 Stack 메모리 소비 | 스레드 오버헤드 적음 | 스레드 오버헤드 없음 |
| 비유 | 팀원 여러 명이 각자 작업 | 소수 직원이 이벤트 큐로 처리 | 1명이 쉬지 않고 위임하며 처리 |

### 어느 쪽이 더 좋은가?

**정답 없다.** 워크로드 특성에 따라 다르다:

- **I/O 집약적 서버** (채팅, 실시간 알림, API Gateway): Node.js 또는 Spring WebFlux 유리
- **CPU 집약적 작업** (데이터 처리, 암호화, ML 추론): Java 멀티스레드 유리
- **일반 CRUD 서비스**: Spring MVC로 충분, 팀 역량/생태계가 더 중요

> Java도 Spring WebFlux를 통해 이벤트루프 방식을 사용할 수 있다. Node.js처럼 Netty 위에서 논블로킹 I/O로 동작한다. → [Spring WebFlux와 이벤트루프](../spring/spring-webflux-event-loop.md)

---

## 4. Java 동기화 메커니즘 심화

멀티스레드에서 Heap(공유 자원)에 동시 접근하면 **Race Condition** 이 발생한다.

### Race Condition이란

```java
// 위험한 코드
class Counter {
    int count = 0;  // Heap에 저장 (공유)

    void increment() {
        count++;  // 이건 사실 3단계 연산
        // 1. count 읽기
        // 2. +1 계산
        // 3. count에 쓰기
        // → 두 스레드가 동시에 1단계를 읽으면 둘 다 같은 값으로 +1해버림
    }
}
```

100번 increment()를 동시 실행하면 100이 아닌 값이 나온다.

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

- 진입 시 **모니터 락(Monitor Lock)** 획득, 종료 시 반납
- 다른 스레드는 락이 반납될 때까지 BLOCKED 상태

```java
// 블록 단위 락 (더 세밀한 제어)
void doSomething() {
    // 여기는 여러 스레드 동시 실행 가능
    prepare();

    synchronized (this) {
        // 이 블록만 한 번에 하나
        count++;
    }

    // 여기도 동시 실행 가능
    after();
}
```

### volatile — 가시성 문제 해결

```
CPU Core 1          CPU Core 2
    ↓                   ↓
L1 Cache            L1 Cache    ← 각 코어가 캐시를 따로 가짐
    ↓                   ↓
           Heap (RAM)
```

캐시로 인해 Thread-1이 값을 변경해도 Thread-2가 오래된 캐시 값을 읽을 수 있다.

```java
// volatile: 캐시 사용 않고 항상 메인 메모리(Heap) 직접 읽기/쓰기
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
| 적합한 상황 | 단순 동기화 | 고급 제어 필요 시 |

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

## 5. Java 멀티스레드 사용 방법 (진화 순서)

### 5-1. Thread 직접 생성 (원시적, 거의 안 씀)

```java
// Runnable 구현 (Thread 상속보다 나음)
Thread t = new Thread(() -> System.out.println("실행: " + Thread.currentThread().getName()));
t.start();
```

- 요청마다 스레드 새로 생성 → **생성/소멸 비용 큼**
- 스레드 수 제어 불가 → 실무에서 거의 미사용

---

### 5-2. ExecutorService — ThreadPool (실무 기본)

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

- 스레드 **재사용**으로 생성 비용 절감
- `future.get()` 이 **블로킹**이라는 게 단점

#### Executors 팩토리 종류

| 팩토리 | 설명 | 주의 |
|---|---|---|
| `newFixedThreadPool(n)` | 고정 스레드 n개 | 큐 무제한 → OOM 가능 |
| `newCachedThreadPool()` | 필요할 때 스레드 생성, 60초 후 소멸 | 스레드 수 무제한 → 과부하 위험 |
| `newSingleThreadExecutor()` | 스레드 1개, 순서 보장 | 처리량 제한 |
| `newScheduledThreadPool(n)` | 스케줄링 (delay, 주기 실행) | — |

> 실무에서는 `Executors` 팩토리보다 `ThreadPoolExecutor`를 직접 설정해 큐 크기를 명시적으로 제한하는 것을 권장한다 (Effective Java).

---

### 5-3. CompletableFuture — 현재 권장 (Java 8+)

```java
// 체이닝
CompletableFuture.supplyAsync(() -> userService.findById(1L))
    .thenApply(user -> user.getName())
    .thenAccept(name -> System.out.println("이름: " + name))
    .exceptionally(e -> { log.error("에러", e); return null; });

// 여러 API 동시 호출 후 합산 (실무 패턴)
CompletableFuture<User> userFuture    = CompletableFuture.supplyAsync(() -> userService.get(id));
CompletableFuture<List<Order>> orderFuture = CompletableFuture.supplyAsync(() -> orderService.getList(id));

CompletableFuture.allOf(userFuture, orderFuture).thenRun(() -> {
    User user        = userFuture.join();
    List<Order> orders = orderFuture.join();
    // 둘 다 완료된 시점에 실행
});
```

#### 주요 메서드 정리

| 메서드 | 설명 |
|---|---|
| `supplyAsync(Supplier)` | 비동기 실행, 반환값 있음 |
| `runAsync(Runnable)` | 비동기 실행, 반환값 없음 |
| `thenApply(Function)` | 결과 변환 (동기) |
| `thenCompose(Function)` | 결과로 새 CompletableFuture 생성 (flatMap) |
| `thenAccept(Consumer)` | 결과 소비, 반환 없음 |
| `allOf(futures...)` | 전부 완료 대기 |
| `anyOf(futures...)` | 하나라도 완료되면 |
| `exceptionally(Function)` | 예외 처리 |
| `join()` | 블로킹 대기 (get()과 유사, unchecked exception) |

> 기본적으로 `ForkJoinPool.commonPool()`을 사용한다. 커스텀 스레드풀을 쓰려면 두 번째 인자로 `Executor`를 전달한다.

---

### 5-4. parallelStream — 간단한 CPU 작업

```java
int sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
```

- **공유 상태가 없는 순수 연산**에만 사용
- 사이드 이펙트 있으면 결과 꼬임 주의

#### 언제 parallelStream이 유리한가

```
유리한 경우:
  - 리스트가 충분히 클 때 (수천 개 이상)
  - 각 원소 처리가 CPU 집약적일 때
  - 순서가 중요하지 않을 때

오히려 느릴 수 있는 경우:
  - 리스트가 작을 때 (스레드 분배 오버헤드가 더 큼)
  - I/O 작업 포함 시 (ForkJoinPool이 I/O에 최적화되지 않음)
  - 순서 보장이 필요할 때 (forEachOrdered 등으로 강제 시 병렬 효과 감소)
```

---

## 6. 상황별 선택 기준

| 상황 | 선택 |
|---|---|
| 여러 외부 API 동시 호출 후 합산 | `CompletableFuture.allOf` |
| 대용량 리스트 CPU 연산 | `parallelStream` |
| Spring 없는 순수 Java 백그라운드 작업 | `ExecutorService` + `CompletableFuture` |
| `new Thread()` 직접 | 거의 쓸 일 없음 |

> Spring 환경에서 백그라운드 작업은 2편의 `@Async`를 참고.

---

**→ 2편: [Spring MVC 요청은 어떤 스레드 위에서 실행되는가](../spring/spring-mvc-thread-safety.md)**

**→ 번외: [Spring WebFlux — Java도 이벤트루프를 쓴다](../spring/spring-webflux-event-loop.md)**
