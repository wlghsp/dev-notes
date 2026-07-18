# Spring이 다 해줬는데 나는 몰랐다 — 어노테이션 뒤에 숨겨진 디자인 패턴들

> Spring을 쓰면서 디자인 패턴을 공부할 기회를 잃었다.
> `@Controller`, `@Service`, `@Transactional`을 붙이는 것만 배웠지,
> 그 안에서 어떤 패턴이 동작하는지는 몰랐다.
> 이 문서는 Spring이 이미 구현해놓은 디자인 패턴들을 꺼내서 직접 마주해보는 기록이다.

---

## 1. Singleton 패턴 — `@Service`, `@Repository`, `@Component`

**Spring이 해준 것**: 모든 Bean을 기본적으로 싱글톤으로 관리한다. `@Service`를 붙이면 애플리케이션 전체에서 인스턴스가 하나만 존재한다.

**패턴 설명**: 클래스의 인스턴스를 오직 하나만 생성하고, 전역에서 접근할 수 있게 한다.

**Spring 없이 Java로 구현한다면**:
```java
public class OrderService {
    // 직접 싱글톤 구현
    private static OrderService instance;

    private OrderService() {}  // 외부 생성 막음

    public static synchronized OrderService getInstance() {
        if (instance == null) {
            instance = new OrderService();
        }
        return instance;
    }
}

// 사용
OrderService service = OrderService.getInstance();
```

**Spring에서는**:
```java
@Service  // 이게 전부. IoC 컨테이너가 싱글톤 생명주기를 관리
public class OrderService { ... }
```

**몰랐던 것**: 싱글톤 Bean은 Heap을 공유하므로 인스턴스 변수에 상태를 저장하면 멀티스레드 환경에서 Race Condition이 발생한다. 편리함 뒤에 이 위험이 숨어 있다.

---

## 2. Proxy 패턴 — `@Transactional`, `@Async`, `@Cacheable`

**Spring이 해준 것**: AOP 프록시로 원본 객체를 감싸 부가 기능(트랜잭션, 캐시, 보안)을 끼워넣는다.

**패턴 설명**: 원본 객체에 접근을 제어하거나 기능을 추가하기 위해 대리 객체(Proxy)를 앞에 세운다.

**Spring 없이 Java로 구현한다면**:
```java
// 인터페이스
interface OrderService {
    void createOrder(Order order);
}

// 원본 구현체
class OrderServiceImpl implements OrderService {
    public void createOrder(Order order) {
        orderRepository.save(order);
    }
}

// 트랜잭션 프록시 직접 구현
class TransactionalOrderServiceProxy implements OrderService {
    private final OrderService target;
    private final TransactionManager tm;

    public TransactionalOrderServiceProxy(OrderService target, TransactionManager tm) {
        this.target = target;
        this.tm = tm;
    }

    @Override
    public void createOrder(Order order) {
        tm.begin();
        try {
            target.createOrder(order);  // 원본 호출
            tm.commit();
        } catch (RuntimeException e) {
            tm.rollback();
            throw e;
        }
    }
}

// 조립
OrderService service = new TransactionalOrderServiceProxy(
    new OrderServiceImpl(), transactionManager
);
```

**Spring에서는**:
```java
@Service
public class OrderServiceImpl {
    @Transactional  // Spring이 프록시를 자동으로 만들어 끼워넣음
    public void createOrder(Order order) {
        orderRepository.save(order);
    }
}
```

**같은 원리로 동작하는 어노테이션들**:

| 어노테이션 | 프록시가 하는 일 |
|---|---|
| `@Transactional` | 트랜잭션 시작/커밋/롤백 |
| `@Async` | 별도 스레드로 실행 위임 |
| `@Cacheable` | 캐시 조회 → 없으면 실행 → 저장 |
| `@PreAuthorize` | 인증/인가 확인 후 실행 |

---

## 3. Template Method 패턴 — `JdbcTemplate`, `RestTemplate`, `KafkaTemplate`

**Spring이 해준 것**: 반복되는 자원 획득/해제, 예외 처리를 Template이 담당하고, 개발자는 핵심 로직만 작성한다.

**패턴 설명**: 알고리즘의 골격을 상위 클래스에 정의하고, 변하는 부분만 하위 클래스(또는 콜백)에서 구현한다.

**Spring 없이 JDBC로 직접 구현한다면**:
```java
// 매번 반복되는 보일러플레이트
Connection conn = null;
PreparedStatement ps = null;
ResultSet rs = null;
try {
    conn = dataSource.getConnection();        // 커넥션 획득
    ps = conn.prepareStatement("SELECT ...");
    ps.setLong(1, id);
    rs = ps.executeQuery();                   // 핵심 로직
    while (rs.next()) { ... }
} catch (SQLException e) {
    throw new RuntimeException(e);            // 예외 변환
} finally {
    if (rs != null) rs.close();              // 자원 해제
    if (ps != null) ps.close();
    if (conn != null) conn.close();
}
```

**Spring JdbcTemplate에서는**:
```java
// 커넥션 획득, 예외 처리, 자원 해제는 Template이 처리
// 개발자는 SQL과 결과 매핑만 작성
Order order = jdbcTemplate.queryForObject(
    "SELECT * FROM orders WHERE id = ?",
    (rs, rowNum) -> new Order(rs.getLong("id"), rs.getString("name")),  // 핵심 로직만
    id
);
```

**같은 패턴이 적용된 것들**:

| 클래스 | Template이 처리하는 것 | 개발자가 작성하는 것 |
|---|---|---|
| `JdbcTemplate` | 커넥션 획득/해제, 예외 변환 | SQL, 결과 매핑 |
| `RestTemplate` | HTTP 연결, 직렬화/역직렬화 | URL, 요청/응답 타입 |
| `KafkaTemplate` | 프로듀서 생성, 전송 확인 | 토픽, 메시지 내용 |
| `TransactionTemplate` | 트랜잭션 시작/커밋/롤백 | 트랜잭션 안에서 실행할 로직 |

---

## 4. Factory 패턴 — `ApplicationContext`, `BeanFactory`

**Spring이 해준 것**: 객체 생성을 컨테이너에 위임한다. 어떤 구현체를 만들지는 설정(어노테이션, XML)이 결정한다.

**패턴 설명**: 객체 생성 로직을 별도의 Factory에 분리해서, 사용하는 쪽이 구체 클래스를 몰라도 되게 한다.

**Spring 없이 직접 구현한다면**:
```java
// 환경에 따라 다른 구현체를 주는 Factory
public class NotificationServiceFactory {
    public static NotificationService create(String env) {
        if ("prod".equals(env)) {
            return new SmsNotificationService();
        } else {
            return new LogNotificationService();  // 테스트용
        }
    }
}

NotificationService service = NotificationServiceFactory.create(System.getenv("ENV"));
```

**Spring에서는**:
```java
// Profile에 따라 다른 Bean을 주입 — Factory 역할을 컨테이너가 담당
@Service
@Profile("prod")
public class SmsNotificationService implements NotificationService { ... }

@Service
@Profile("!prod")
public class LogNotificationService implements NotificationService { ... }

// 사용하는 쪽은 구현체를 전혀 모름
@Autowired
private NotificationService notificationService;  // Spring이 알아서 골라줌
```

---

## 5. Observer 패턴 — `ApplicationEvent`, `@EventListener`

**Spring이 해준 것**: 이벤트 발행/구독 메커니즘. 발행자와 구독자가 서로를 모른다.

**패턴 설명**: 상태 변화가 발생하면 이를 구독하는 객체들에게 자동으로 알린다. 발행자와 구독자 사이의 결합도를 낮춘다.

**Spring 없이 직접 구현한다면**:
```java
// Observer 인터페이스
interface OrderEventListener {
    void onOrderCreated(Order order);
}

// Subject (발행자)
class OrderService {
    private List<OrderEventListener> listeners = new ArrayList<>();

    public void addListener(OrderEventListener listener) {
        listeners.add(listener);
    }

    public void createOrder(Order order) {
        orderRepository.save(order);
        // 모든 구독자에게 직접 알림
        for (OrderEventListener listener : listeners) {
            listener.onOrderCreated(order);
        }
    }
}

// 구독자 등록
OrderService orderService = new OrderService();
orderService.addListener(new EmailNotificationListener());
orderService.addListener(new StockDeductionListener());
```

**Spring에서는**:
```java
// 이벤트 정의
public class OrderCreatedEvent {
    private final Order order;
    public OrderCreatedEvent(Order order) { this.order = order; }
}

// 발행자 — 구독자가 누구인지 전혀 모름
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public void createOrder(Order order) {
        orderRepository.save(order);
        eventPublisher.publishEvent(new OrderCreatedEvent(order));  // 발행만
    }
}

// 구독자들 — 발행자가 누구인지 전혀 모름
@Component
public class EmailNotificationListener {
    @EventListener
    public void handle(OrderCreatedEvent event) {
        emailService.send(event.getOrder());
    }
}

@Component
public class StockDeductionListener {
    @EventListener
    public void handle(OrderCreatedEvent event) {
        stockService.deduct(event.getOrder());
    }
}
```

**왜 쓰는가**: 주문 생성 시 이메일 발송, 재고 차감, 포인트 적립 등 여러 작업이 필요할 때, `OrderService`가 모든 서비스를 직접 의존하지 않아도 된다. 새 구독자를 추가해도 `OrderService`는 건드리지 않는다.

---

## 6. Decorator 패턴 — `HandlerInterceptor`, `Filter`

**Spring이 해준 것**: 요청/응답 처리에 기능을 덧씌운다. 원본 컨트롤러 코드를 건드리지 않고 로깅, 인증, CORS 처리를 추가한다.

**패턴 설명**: 원본 객체를 감싸서 기능을 동적으로 추가한다. 상속 없이 책임을 확장한다.

**Spring 없이 직접 구현한다면**:
```java
// 인터페이스
interface RequestHandler {
    void handle(Request req, Response res);
}

// 원본
class OrderController implements RequestHandler {
    public void handle(Request req, Response res) {
        // 비즈니스 로직
    }
}

// 로깅 Decorator
class LoggingHandler implements RequestHandler {
    private final RequestHandler target;

    public void handle(Request req, Response res) {
        log.info("요청: {}", req.getPath());
        target.handle(req, res);               // 원본 실행
        log.info("응답 완료");
    }
}

// 인증 Decorator
class AuthHandler implements RequestHandler {
    private final RequestHandler target;

    public void handle(Request req, Response res) {
        if (!isAuthenticated(req)) throw new UnauthorizedException();
        target.handle(req, res);
    }
}

// 조립: 겹겹이 감쌈
RequestHandler handler = new LoggingHandler(
    new AuthHandler(
        new OrderController()
    )
);
```

**Spring에서는**:
```java
// Filter (Servlet 레벨) — 가장 바깥
@Component
public class LoggingFilter extends OncePerRequestFilter {
    protected void doFilterInternal(HttpServletRequest req, ...) {
        log.info("요청: {}", req.getRequestURI());
        filterChain.doFilter(req, res);  // 다음 레이어로
    }
}

// HandlerInterceptor (Spring MVC 레벨) — 그 다음
@Component
public class AuthInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest req, ...) {
        if (!isAuthenticated(req)) throw new UnauthorizedException();
        return true;
    }
}

// Controller — 가장 안쪽. 위 두 레이어를 전혀 모름
@RestController
public class OrderController {
    @GetMapping("/orders")
    public List<Order> getOrders() { ... }
}
```

---

## 7. Strategy 패턴 — `@Qualifier`, 인터페이스 다형성

**Spring이 해준 것**: 같은 인터페이스의 여러 구현체를 Bean으로 등록하고 상황에 따라 교체한다.

**패턴 설명**: 알고리즘(전략)을 인터페이스로 추상화하고, 런타임에 구현체를 교체할 수 있게 한다.

**Spring 없이 직접 구현한다면**:
```java
interface PaymentStrategy {
    void pay(int amount);
}

class CardPayment implements PaymentStrategy {
    public void pay(int amount) { /* 카드 결제 */ }
}

class KakaoPayPayment implements PaymentStrategy {
    public void pay(int amount) { /* 카카오페이 결제 */ }
}

// 전략 선택
class OrderService {
    private PaymentStrategy strategy;

    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void createOrder(Order order) {
        strategy.pay(order.getAmount());  // 어떤 전략인지 모름
    }
}
```

**Spring에서는**:
```java
@Component("card")
public class CardPayment implements PaymentStrategy { ... }

@Component("kakaopay")
public class KakaoPayPayment implements PaymentStrategy { ... }

@Service
public class OrderService {
    // @Qualifier로 전략 선택
    @Autowired @Qualifier("kakaopay")
    private PaymentStrategy paymentStrategy;

    // 또는 Map으로 전체 주입 후 런타임에 선택
    @Autowired
    private Map<String, PaymentStrategy> strategies;

    public void createOrder(Order order, String method) {
        strategies.get(method).pay(order.getAmount());
    }
}
```

---

## 8. 정리 — Spring 어노테이션과 디자인 패턴 대응표

| Spring 기능 | 디자인 패턴 | 핵심 목적 |
|---|---|---|
| `@Service`, `@Component` | Singleton | 인스턴스 하나, 전역 공유 |
| `@Transactional`, `@Async`, `@Cacheable` | Proxy | 원본 수정 없이 기능 추가 |
| `JdbcTemplate`, `RestTemplate` | Template Method | 반복 코드 제거, 핵심 로직 위임 |
| `ApplicationContext`, `@Profile` | Factory | 객체 생성 로직 분리 |
| `ApplicationEvent`, `@EventListener` | Observer | 느슨한 결합, 이벤트 기반 |
| `Filter`, `HandlerInterceptor` | Decorator | 기능을 겹겹이 추가 |
| `@Qualifier`, 인터페이스 다형성 | Strategy | 알고리즘 교체 가능 |

### 왜 이걸 알아야 하는가

Spring이 이 패턴들을 대신 구현해줬기 때문에 우리는 생산성을 얻었다. 하지만 그 대가로 **패턴을 배울 기회**를 잃었다.

직접 구현해본 적 없는 패턴은 장애 상황에서 원인 파악이 어렵다. `@Transactional`이 왜 같은 클래스 내부 호출에서 동작하지 않는지, `@Async`가 왜 별도 Bean에서만 동작하는지 — 이 모든 게 Proxy 패턴을 이해하면 단번에 납득된다.

> **어노테이션을 외우는 것과 패턴을 이해하는 것은 다르다.**
> 패턴을 알면 Spring의 동작이 예측 가능해지고, 문제가 생겼을 때 어디를 봐야 하는지 알 수 있다.

---

**관련 문서**
- [Spring이 다 해줬는데 나는 몰랐다 — 편리함 뒤에 숨겨진 것들](spring-hidden-internals.md)
- [Spring 개발자인데 멀티스레드를 몰랐다 (2편)](spring-mvc-thread-safety.md)
