# Spring이 다 해줬는데 나는 몰랐다 — 편리함 뒤에 숨겨진 것들

> Spring은 너무 편하다. 그런데 그 편함이 개발자의 이해를 조용히 갉아먹는다.
> 다른 언어/프레임워크에서는 직접 구현해야 하는 것들을 Spring은 어노테이션 하나로 끝낸다.
> 그 결과, 우리는 동작은 알지만 **왜 동작하는지**를 모르는 개발자가 되어간다.
> 이 문서는 그 "모르고 지나친 것들"을 하나씩 꺼내보는 시리즈다.

---

## 1. 멀티스레드 — 나는 단일 스레드인 줄 알았다

**Spring이 해준 것**: Tomcat ThreadPool. 요청마다 스레드를 꺼내 DispatcherServlet에 넘긴다.

**내가 몰랐던 것**: 내가 작성한 `@Service` 메서드는 동시에 200개 스레드에서 실행될 수 있다.

```java
@Service
public class OrderService {
    private int orderCount = 0;  // ❌ 이게 왜 위험한지 몰랐다

    public void createOrder() {
        orderCount++;  // 200개 스레드가 동시에 접근 → Race Condition
    }
}
```

**다른 언어에서는**: Node.js Express는 싱글스레드라 이 문제가 없다. Go는 goroutine을 직접 다루며 `sync.Mutex`를 명시적으로 쓴다. Java는 스레드 관리를 Tomcat이 대신하다 보니 오히려 의식하기 어렵다.

**Spring 없이 Java로 구현한다면**:
```java
// 직접 ThreadPool 만들고 요청마다 스레드 할당
ExecutorService pool = new ThreadPoolExecutor(
    10, 200, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1000)
);
ServerSocket server = new ServerSocket(8080);
while (true) {
    Socket socket = server.accept();
    pool.submit(() -> handleRequest(socket));  // 직접 라우팅도 구현해야 함
}
```

→ 자세한 내용은 [멀티스레드 1편](java-multithread-pt1.md), [2편](java-multithread-pt2.md) 참고

---

## 2. 트랜잭션 — `@Transactional` 뒤에서 일어나는 일

**Spring이 해준 것**: AOP 프록시로 트랜잭션 시작/커밋/롤백을 자동 처리.

**내가 몰랐던 것**: `@Transactional`은 어노테이션이 아니라 **프록시 객체 교체**다. Spring이 내 Bean을 감싸는 프록시를 만들어 끼워넣는다.

```java
// 내가 쓴 코드
@Service
public class OrderService {
    @Transactional
    public void createOrder(Order order) {
        orderRepository.save(order);
        paymentRepository.save(payment);
    }
}

// Spring이 실제로 만드는 것 (개념적으로)
public class OrderService$$SpringProxy extends OrderService {
    @Override
    public void createOrder(Order order) {
        TransactionManager.begin();       // ① 트랜잭션 시작
        try {
            super.createOrder(order);     // ② 내 코드 실행
            TransactionManager.commit();  // ③ 커밋
        } catch (RuntimeException e) {
            TransactionManager.rollback(); // ④ 롤백
            throw e;
        }
    }
}
```

**이래서 발생하는 함정**:
```java
@Service
public class OrderService {
    public void outer() {
        this.inner();  // ❌ 프록시를 거치지 않음 → @Transactional 무시됨
    }

    @Transactional
    public void inner() {
        // 트랜잭션이 시작되지 않는다
    }
}
```

**다른 언어에서는**: Python Django는 `with transaction.atomic():` 블록으로 명시적으로 감싼다. Go는 `db.Begin()` → `tx.Commit()` / `tx.Rollback()`을 직접 호출한다. 명시적이라 동작이 눈에 보인다.

**Spring 없이 Java로 구현한다면**:
```java
Connection conn = dataSource.getConnection();
conn.setAutoCommit(false);
try {
    orderDao.save(conn, order);
    paymentDao.save(conn, payment);
    conn.commit();
} catch (Exception e) {
    conn.rollback();
    throw e;
} finally {
    conn.close();
}
```

---

## 3. 의존성 주입(DI) — 객체를 내가 만들지 않는다

**Spring이 해준 것**: IoC 컨테이너가 Bean을 생성하고 주입한다.

**내가 몰랐던 것**: `@Autowired`는 마법이 아니라 **리플렉션으로 필드에 객체를 밀어넣는 것**이다.

```java
// 내가 쓴 코드
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    // orderRepository가 어디서 왔는지 모른 채 그냥 씀
}

// Spring이 실제로 하는 것 (개념적으로)
OrderService service = new OrderService();
Field field = OrderService.class.getDeclaredField("orderRepository");
field.setAccessible(true);
field.set(service, new OrderRepository()); // 리플렉션으로 주입
```

**다른 언어에서는**: Go는 DI 프레임워크 없이 생성자에 직접 넘긴다. Python도 마찬가지다. 객체 생성 흐름이 코드에 그대로 드러난다.

**Spring 없이 Java로 구현한다면**:
```java
// 직접 조립
DataSource dataSource = new HikariDataSource(config);
OrderRepository orderRepository = new OrderRepository(dataSource);
OrderService orderService = new OrderService(orderRepository);
OrderController orderController = new OrderController(orderService);
// 이걸 모든 Bean마다 해야 함
```

> Spring DI의 진짜 가치는 "객체 그래프 조립의 자동화"다. 편리함을 누리되, 내부가 리플렉션 기반이라는 것, 그래서 컴파일 타임에 오류를 못 잡는다는 것을 알아야 한다.

---

## 4. 커넥션 풀 — DB 연결을 매번 새로 만들지 않는다

**Spring이 해준 것**: HikariCP가 DB 커넥션을 미리 만들어 풀로 관리한다.

**내가 몰랐던 것**: `repository.save()`를 호출할 때마다 DB 연결이 새로 생기는 게 아니다. **풀에서 빌려 쓰고 반납**한다.

```
[커넥션 풀 동작]

애플리케이션 시작 → HikariCP가 커넥션 10개 미리 생성 (기본값)

요청 A: repository.save() → 풀에서 Connection-1 빌림 → 완료 후 반납
요청 B: repository.save() → 풀에서 Connection-2 빌림 → 완료 후 반납

풀이 고갈되면 → 다음 요청은 반납될 때까지 대기 (Connection Timeout)
```

**이래서 발생하는 문제**:
```java
@Transactional
public void createOrder() {
    orderRepository.save(order);

    // 여기서 외부 API 호출 (수 초 소요)
    externalApiClient.call();   // ← 이 동안 커넥션을 붙잡고 있음

    paymentRepository.save(payment);
}
```

트랜잭션 안에서 외부 API를 호출하면 커넥션을 점유한 채 대기하게 된다. 동시 요청이 많으면 **커넥션 풀 고갈 → 전체 서비스 지연**.

**다른 언어에서는**: Python의 SQLAlchemy, Go의 `database/sql` 모두 커넥션 풀을 명시적으로 설정한다. `db.SetMaxOpenConns(10)` 같은 코드가 직접 드러난다.

**Spring 없이 Java로 구현한다면**:
```java
// JDBC만 쓰면 매 요청마다 새 연결 (매우 느림)
Connection conn = DriverManager.getConnection(url, user, password);
// 연결 생성: 수십 ms ~ 수백 ms 소요
```

---

## 5. AOP — 공통 로직이 어떻게 모든 메서드에 끼어드는가

**Spring이 해준 것**: 프록시 기반 AOP로 로깅, 보안, 트랜잭션을 횡단 관심사로 분리.

**내가 몰랐던 것**: `@Transactional`, `@PreAuthorize`, `@Cacheable` 모두 **AOP 프록시가 메서드 호출을 가로채는 것**이다. 같은 원리다.

```java
// 이 세 어노테이션은 모두 같은 메커니즘
@Transactional        // → 트랜잭션 AOP
@PreAuthorize("...")  // → 보안 AOP
@Cacheable("orders")  // → 캐시 AOP
public Order getOrder(Long id) { ... }

// 실제 호출 흐름
// 캐시 확인 → 인증/인가 확인 → 트랜잭션 시작 → 내 코드 → 트랜잭션 커밋 → 캐시 저장
```

**다른 언어에서는**: Python은 데코레이터(`@`)로 직접 구현한다. 동작이 코드에 명시적으로 드러난다.
```python
def transactional(func):
    def wrapper(*args, **kwargs):
        db.begin()
        try:
            result = func(*args, **kwargs)
            db.commit()
            return result
        except:
            db.rollback()
            raise
    return wrapper

@transactional
def create_order(order): ...
```

**같은 클래스 내부 호출이 AOP를 우회하는 이유**:
```
외부에서 호출: Client → [Spring Proxy] → 내 메서드  ← AOP 동작
내부에서 호출: 내 메서드 → this.다른메서드()        ← Proxy 우회, AOP 무시
```

---

## 6. 직렬화/역직렬화 — JSON이 자동으로 객체가 되는 원리

**Spring이 해준 것**: Jackson이 HTTP 요청 Body의 JSON을 Java 객체로, 응답 객체를 JSON으로 자동 변환.

**내가 몰랐던 것**: `@RequestBody`와 `@ResponseBody`는 **Jackson ObjectMapper가 리플렉션으로 필드를 읽어 변환**하는 것이다.

```java
// 이게 동작하려면 기본 생성자 + getter/setter (또는 @JsonProperty) 가 필요
@PostMapping("/orders")
public OrderResponse createOrder(@RequestBody OrderRequest request) {
    // request는 어떻게 JSON에서 왔는가?
}

// Jackson이 하는 일 (개념적으로)
String json = "{\"itemId\": 1, \"quantity\": 2}";
OrderRequest request = new OrderRequest();  // 기본 생성자 필요
Field field = OrderRequest.class.getDeclaredField("itemId");
field.set(request, 1);  // 리플렉션으로 값 세팅
```

**이래서 발생하는 함정**:
```java
public class OrderRequest {
    private Long itemId;
    // ❌ 기본 생성자 없음, getter 없음 → Jackson 역직렬화 실패
    public OrderRequest(Long itemId) {
        this.itemId = itemId;
    }
}
```

---

## 7. 정리 — Spring이 숨긴 것들의 공통점

| Spring 기능 | 내부 메커니즘 | 모르면 생기는 문제 |
|---|---|---|
| `@Transactional` | AOP 프록시 | 같은 클래스 내부 호출 시 트랜잭션 무시 |
| `@Autowired` | 리플렉션 DI | 순환 의존성, 테스트 어려움 |
| Tomcat ThreadPool | 요청당 스레드 할당 | 싱글톤 Bean 인스턴스 변수 Race Condition |
| HikariCP | 커넥션 풀 재사용 | 트랜잭션 안 외부 API 호출로 풀 고갈 |
| `@Cacheable` | AOP + 캐시 저장소 | 같은 클래스 내부 호출 시 캐시 무시 |
| `@RequestBody` | Jackson 리플렉션 | 기본 생성자 없으면 역직렬화 실패 |

### 왜 이걸 알아야 하는가

Spring의 추상화는 훌륭하다. 생산성을 높여주고 반복 코드를 없애준다. 하지만 **추상화는 문제를 숨기는 게 아니라 미루는 것**이다. 장애가 났을 때, 성능 문제가 생겼을 때, 테스트가 안 될 때 — 그때 내부를 모르면 원인을 찾지 못한다.

다른 언어/프레임워크를 보면 Spring이 당연하게 해주는 것들이 사실은 상당한 구현을 감추고 있다는 걸 알 수 있다. 그 감춰진 부분을 하나씩 꺼내보는 게 이 시리즈의 목적이다.

> **"프레임워크를 쓰는 것"과 "프레임워크를 이해하고 쓰는 것"은 다르다.**
