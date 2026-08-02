# Project Loom

## 왜 등장했나

Java에서 동시 요청을 처리하는 전통적인 방법은 스레드풀이다. 요청 하나당 스레드 하나. 그런데 스레드는 비싸다.

OS 스레드 하나는 약 1MB의 스택 메모리를 차지한다. 10,000개의 동시 요청 = 10GB 메모리. 현실적으로 수백~수천 개가 한계다.

I/O 대기 시간이 문제다. DB 쿼리 날리고 응답 기다리는 동안 스레드는 아무것도 안 하면서 자리만 차지한다. CPU 사용률이 낮아도 스레드가 부족해서 요청을 못 받는 상황이 생긴다.

이걸 해결하려고 Reactive Programming(WebFlux 등)이 나왔다. 비동기 논블로킹으로 스레드를 효율적으로 쓰지만, 코드가 복잡해진다. `flatMap`, `subscribe`, 콜백 지옥. 디버깅이 어렵고 기존 블로킹 라이브러리(JDBC 등)와 섞이면 문제가 생긴다.

**Project Loom은 다른 접근이다. 블로킹 코드를 그대로 쓰면서, JVM이 내부에서 비동기처럼 동작하게 만든다.**

---

## 핵심: Virtual Thread

Java 21에서 정식 출시. 기존 OS 스레드(Platform Thread)와 별개로, JVM이 직접 관리하는 경량 스레드다.

**Platform Thread (기존):**
- OS 스레드와 1:1 대응
- 스택 메모리 ~1MB
- 생성/컨텍스트 스위칭 비용 큼
- 수천 개가 실용적 한계

**Virtual Thread:**
- JVM이 관리, OS 스레드와 1:1 대응 없음
- 힙에 할당, 스택 ~수 KB (필요 시 동적 확장)
- 생성 비용이 거의 없음
- 수백만 개도 가능

```java
// Platform Thread
Thread t = new Thread(runnable);

// Virtual Thread
Thread vt = Thread.ofVirtual().start(runnable);

// ExecutorService로
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest());
}
```

---

## 동작 원리: Carrier Thread와 Unmounting

Virtual Thread는 **Carrier Thread** (실제 OS 스레드) 위에서 실행된다.

```
OS Thread (Carrier Thread) ──── Virtual Thread A 실행
                           ──── Virtual Thread A: DB 쿼리 대기 (I/O 블록)
                                     |
                                JVM이 Virtual Thread A를 Carrier에서 분리(unmount)
                                     |
                           ──── Virtual Thread B 실행 (같은 Carrier Thread 재사용)
                                     |
                                DB 응답 도착 → Virtual Thread A 재개(mount)
```

블로킹 I/O가 발생하면 JVM이 자동으로 해당 Virtual Thread의 스택 상태를 힙에 저장하고, Carrier Thread를 다른 Virtual Thread에게 넘긴다. I/O 완료 후 다시 Carrier에 올려서 실행을 재개한다.

개발자는 블로킹 코드를 그대로 쓴다. JVM이 알아서 처리한다.

---

## 주의: Thread Pinning

Virtual Thread가 `synchronized` 블록 안에 있거나 native method를 호출하면, Carrier Thread에서 **분리가 불가능**해진다. 이 상태를 **Pinning**이라고 한다.

Pinning이 발생하면 Virtual Thread는 Carrier Thread를 독점한다. Carrier가 다른 Virtual Thread를 실행하지 못하므로, 그냥 Platform Thread처럼 동작하게 된다.

```java
// Pinning이 발생하는 코드
synchronized (lock) {
    Thread.sleep(1000);  // 여기서 Carrier 독점
}

// Pinning 없이 쓰려면
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    Thread.sleep(1000);  // 여기서 Carrier 분리 가능
} finally {
    lock.unlock();
}
```

JDK 24(JEP 491)에서 `synchronized` 블록 내 Pinning 문제가 대부분 해결됐다.

Pinning 여부는 JVM 플래그로 감지할 수 있다.
```
-Djdk.tracePinnedThreads=full
```

---

## CPU 집약적 작업에는 안 어울린다

Virtual Thread는 I/O 대기가 많은 작업에 최적화돼 있다.

CPU를 지속적으로 사용하는 계산 작업(이미지 처리, 암호화, 복잡한 연산)은 Virtual Thread를 써도 OS 스레드 수 이상으로 병렬도가 올라가지 않는다. Carrier Thread (OS 스레드) 수가 병목이 되고, Virtual Thread의 장점이 없다.

I/O 대기가 있어야 Carrier Thread를 효율적으로 나눠 쓸 수 있다.

---

## Spring Boot와의 통합

Spring Boot 3.2부터 한 줄로 활성화된다.

```properties
# application.properties
spring.threads.virtual.enabled=true
```

이 설정 하나로 Tomcat의 요청 처리 스레드가 Virtual Thread로 전환된다. 기존 코드 수정 없이 동작한다.

Spring MVC (블로킹 방식) + Virtual Thread 조합이면, WebFlux 없이도 높은 동시성을 처리할 수 있다. JDBC, JPA 같은 블로킹 라이브러리도 그대로 쓸 수 있다.

Reactive의 복잡한 코드 스타일 없이, 단순한 블로킹 코드로 Reactive 수준의 처리량을 목표로 한다.

---

## Structured Concurrency (구조적 동시성)

Project Loom이 함께 가져온 개념. Java 21 Preview, Java 25에서 Stable 예정.

기존 멀티스레드 코드의 문제: 스레드를 독립적으로 실행하면 에러 처리, 취소, 리소스 정리가 복잡해진다.

```java
// 기존 방식: 스레드 하나 실패해도 다른 게 계속 돌 수 있음
Future<String> userFuture = executor.submit(() -> fetchUser(id));
Future<String> orderFuture = executor.submit(() -> fetchOrders(id));
// 하나 실패하면? 나머지 취소하는 로직 직접 짜야 함
```

Structured Concurrency는 스코프를 명확히 한다. 스코프가 닫힐 때 모든 하위 태스크가 완료되거나 취소된다.

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> user   = scope.fork(() -> fetchUser(id));
    Subtask<String> orders = scope.fork(() -> fetchOrders(id));

    scope.join();           // 둘 다 완료될 때까지 대기
    scope.throwIfFailed();  // 하나라도 실패하면 예외

    // 여기 도달하면 둘 다 성공
    return new Response(user.get(), orders.get());
}
// 스코프 종료 시 아직 실행 중인 태스크는 자동 취소
```

---

## 요약

- Platform Thread는 OS 스레드와 1:1 대응, 수천 개가 한계
- Virtual Thread는 JVM 관리, 수백만 개도 가능, 스택은 ~수 KB
- I/O 블록 시 JVM이 자동으로 Carrier Thread에서 분리(unmount)
- `synchronized` + 블로킹은 Pinning 발생 주의 (`ReentrantLock` 권장)
- CPU 집약적 작업에는 효과 없음
- Spring Boot 3.2+에서 `spring.threads.virtual.enabled=true` 한 줄로 활성화
- Structured Concurrency로 태스크 생명주기 관리를 구조화할 수 있음
