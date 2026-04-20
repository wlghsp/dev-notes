# Spring WebFlux가 숨긴 것들 — Java도 이벤트루프를 쓴다

> Spring MVC만 쓰다 보면 "Java = 멀티스레드"로 굳어버린다. 그런데 Spring WebFlux는 Node.js처럼 이벤트루프로 동작한다. 같은 JVM 위에서 어떻게 가능한가.

---

## 1. Spring MVC vs Spring WebFlux — 무엇이 다른가

```
Spring MVC (전통적)
  요청 → Tomcat → 스레드 할당 → Controller → 응답
  스레드가 I/O 동안 블로킹 대기

Spring WebFlux (리액티브)
  요청 → Netty → 이벤트루프 스레드 → Handler → 응답
  I/O는 논블로킹, 스레드를 점유하지 않음
```

| 항목 | Spring MVC | Spring WebFlux |
|---|---|---|
| 서버 | Tomcat (기본) | Netty (기본) |
| 스레드 모델 | 요청당 1스레드 | 이벤트루프 (소수 스레드) |
| I/O | 블로킹 | 논블로킹 |
| 반환 타입 | `String`, `ResponseEntity` | `Mono<T>`, `Flux<T>` |
| 학습 난이도 | 낮음 | 높음 |

---

## 2. 내부 구조 — Netty 이벤트루프

Spring WebFlux의 기반은 **Netty**다.

```
HTTP 요청
    ↓
Netty EventLoop (소수 스레드, 보통 CPU 코어 수 × 2)
    ↓
Channel Pipeline (Handler 체인)
    ↓
WebFlux DispatcherHandler
    ↓
Controller 반환: Mono<T> / Flux<T>
    ↓
I/O 완료 콜백 → 같은 EventLoop에서 처리
```

- Netty는 **이벤트루프 스레드 풀**을 운영한다 (보통 코어 수 × 2개)
- 각 스레드는 여러 Channel(연결)을 **비동기**로 처리
- I/O 대기 동안 스레드를 놀리지 않고 다른 Channel 처리

> Node.js와 차이: Node.js는 이벤트루프 스레드가 **1개**, Netty는 **여러 개**. JVM은 멀티코어를 활용하면서도 논블로킹이다.

---

## 3. Mono와 Flux — 리액티브 타입

Spring WebFlux에서 컨트롤러는 즉시 값을 반환하지 않고 **"나중에 값이 올 것"** 이라는 약속을 반환한다.

```java
// Spring MVC — 블로킹, 값을 직접 반환
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id);  // DB 응답까지 스레드 대기
}

// Spring WebFlux — 논블로킹, 약속을 반환
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return userRepository.findById(id);  // 즉시 Mono 반환, 스레드 해방
}
```

| 타입 | 의미 | 비유 |
|---|---|---|
| `Mono<T>` | 0 또는 1개의 값 | `Optional` + 비동기 |
| `Flux<T>` | 0개 이상의 값 스트림 | `List` + 비동기 |

### 체이닝

```java
// 여러 비동기 작업을 체이닝
public Mono<UserResponse> getUserWithOrders(Long userId) {
    return userRepository.findById(userId)          // Mono<User>
        .flatMap(user ->
            orderRepository.findByUserId(user.getId())  // Mono<List<Order>>
                .map(orders -> new UserResponse(user, orders))
        )
        .onErrorResume(e -> Mono.just(UserResponse.empty()));
}
```

### 여러 API 동시 호출

```java
// 두 API를 동시에 호출하고 결과 합산
public Mono<DashboardData> getDashboard(Long userId) {
    Mono<User> userMono   = userService.findById(userId);
    Mono<List<Order>> orderMono = orderService.findByUserId(userId);

    return Mono.zip(userMono, orderMono)
        .map(tuple -> new DashboardData(tuple.getT1(), tuple.getT2()));
}
```

---

## 4. 스케줄러 — 블로킹 코드 다루기

이벤트루프 스레드에서 **블로킹 코드를 실행하면 전체 서버가 멈춘다.** Node.js와 같은 문제다.

```java
// 위험: 이벤트루프 스레드에서 블로킹
public Mono<String> bad() {
    return Mono.fromCallable(() -> {
        Thread.sleep(1000);  // 이벤트루프 스레드 점유 → 다른 요청 처리 불가
        return "result";
    });
}

// 올바름: 블로킹 작업은 별도 스케줄러로 위임
public Mono<String> good() {
    return Mono.fromCallable(() -> {
        Thread.sleep(1000);
        return "result";
    }).subscribeOn(Schedulers.boundedElastic());  // 블로킹 전용 스레드풀
}
```

| 스케줄러 | 용도 |
|---|---|
| `Schedulers.parallel()` | CPU 집약 작업 (코어 수만큼 스레드) |
| `Schedulers.boundedElastic()` | 블로킹 I/O (동적으로 스레드 생성, 상한 있음) |
| `Schedulers.single()` | 단일 스레드, 순서 보장 |
| `Schedulers.fromExecutor(e)` | 커스텀 스레드풀 |

---

## 5. Node.js 이벤트루프와 비교

```
Node.js                         Spring WebFlux (Netty)
─────────────────────────────   ──────────────────────────────
이벤트루프 스레드: 1개           이벤트루프 스레드: CPU 코어 × 2
libuv → OS 비동기 I/O           Netty NIO → OS 비동기 I/O
콜백 / Promise / async-await    Mono / Flux / 체이닝
CPU 블로킹 → 전체 서버 멈춤      이벤트루프 블로킹 → 전체 서버 멈춤
Worker Threads로 CPU 작업 위임   Schedulers.parallel()로 CPU 작업 위임
```

구조적으로 같은 문제를 같은 방식으로 푼다. **이벤트루프를 블로킹하지 마라.**

---

## 6. Spring MVC vs WebFlux — 언제 무엇을 선택하는가

```
WebFlux가 유리한 경우:
  - 동시 연결이 매우 많은 서비스 (채팅, 실시간 알림, API Gateway)
  - 여러 외부 API/DB를 동시에 호출하는 서비스
  - 스트리밍 응답 (Server-Sent Events, WebSocket)

Spring MVC로 충분한 경우:
  - 일반 CRUD 서비스
  - 팀이 리액티브 패턴에 익숙하지 않을 때
  - JPA 등 블로킹 라이브러리를 주로 사용할 때
    (블로킹 라이브러리를 WebFlux에서 쓰면 오히려 복잡도만 증가)
```

> WebFlux는 도구다. 리액티브 패턴의 학습 비용을 정당화할 만큼 I/O 동시성 문제가 있을 때 선택한다. 없다면 Spring MVC가 더 단순하고 유지보수하기 쉽다.

---

**← 1편: [Java 멀티스레드 vs Node.js 이벤트루프](../java/java-vs-nodejs-thread-model.md)**

**← 2편: [Spring MVC 요청은 어떤 스레드 위에서 실행되는가](./spring-mvc-thread-safety.md)**
