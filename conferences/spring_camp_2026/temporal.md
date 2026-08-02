# Temporal

## 한 줄 요약

**"서버가 죽어도 워크플로우가 멈추지 않는다"** — 분산 시스템에서 장기 실행 비즈니스 플로우를 안전하게 처리하는 워크플로우 오케스트레이션 엔진.

---

## 왜 필요한가

마이크로서비스 환경에서 주문 하나를 처리하는 흐름을 생각해보자.

```
주문 접수 → 결제 처리 → 재고 차감 → 배송 등록 → 알림 발송
```

결제는 성공했는데 재고 차감 도중 서버가 죽으면? 결제는 됐는데 주문은 실패 상태가 된다. 이 불일치를 어떻게 처리할까.

전통적인 해법들:
- **분산 트랜잭션(2PC)**: 모든 서비스가 락을 잡고 동시에 커밋. 느리고 가용성이 떨어짐
- **Saga 패턴**: 각 단계마다 보상 트랜잭션을 직접 구현. 코드가 복잡해짐
- **직접 재시도 로직**: 멱등성 보장, 상태 저장, 타임아웃 처리를 개발자가 모두 구현

Temporal은 이 문제를 **인프라 레벨에서 해결**한다. 개발자는 비즈니스 로직만 코드로 쓰면 된다.

---

## 핵심 개념 세 가지

### Workflow

비즈니스 플로우의 전체 흐름을 정의하는 코드다. 순서, 조건 분기, 재시도 정책을 담는다.

```java
@WorkflowInterface
public interface OrderWorkflow {
    @WorkflowMethod
    void processOrder(Order order);
}

public class OrderWorkflowImpl implements OrderWorkflow {

    private final PaymentActivity payment =
        Workflow.newActivityStub(PaymentActivity.class, options);
    private final InventoryActivity inventory =
        Workflow.newActivityStub(InventoryActivity.class, options);

    @Override
    public void processOrder(Order order) {
        payment.charge(order.getAmount());      // 결제
        inventory.deduct(order.getItemId());    // 재고 차감
        // 서버가 여기서 죽어도 자동으로 재개됨
    }
}
```

**핵심 제약: Workflow 코드는 반드시 결정적(Deterministic)이어야 한다.**
같은 입력이면 항상 같은 순서로 실행되어야 한다. `Math.random()`, `System.currentTimeMillis()` 같은 비결정적 코드를 Workflow 안에서 쓰면 안 된다. (Activity 안에서는 가능)

### Activity

실제 작업을 수행하는 단위. 외부 API 호출, DB 쓰기, 이메일 발송 등 부수 효과가 있는 작업은 여기에 담는다.

```java
@ActivityInterface
public interface PaymentActivity {
    void charge(long amount);
}

public class PaymentActivityImpl implements PaymentActivity {
    @Override
    public void charge(long amount) {
        // 실제 결제 API 호출
        // 실패하면 Temporal이 자동으로 재시도
    }
}
```

Activity는 재시도, 타임아웃, 에러 처리를 Temporal이 담당한다. 개발자는 비즈니스 로직만 구현.

### Worker

Workflow와 Activity 코드를 실제로 실행하는 프로세스. Temporal 서버에 폴링해서 할당된 태스크를 처리한다.

```
[Temporal 서버] ← 폴링 ← [Worker 프로세스 (Spring Boot 앱)]
                → 태스크 →
```

Worker가 죽으면 다른 Worker가 이어받는다. 실행 상태는 Temporal 서버에 저장돼 있으므로 처음부터 다시 시작하지 않는다.

---

## Temporal Server와 Worker의 역할 분리

**Temporal Server** — 두뇌, 상태 저장소
- Workflow 실행 상태, Event History를 DB에 저장
- Worker에게 태스크를 배분
- 직접 비즈니스 로직을 실행하지 않음
- Temporal이 제공하는 인프라 (self-host 또는 Temporal Cloud)

**Worker** — 손발, 실제 실행자
- 개발자가 짠 Spring Boot 앱 안에 함께 뜨는 프로세스
- Temporal Server에 계속 폴링 ("나 할 일 있어?")
- 태스크 받으면 Workflow / Activity 코드 실행
- **Stateless** — 실행만 하고 상태를 직접 가지지 않음

Worker가 Stateless인 덕분에 여러 개를 띄워도 된다. 어떤 Worker가 처리하든 Temporal Server의 Event History만 있으면 이어서 실행할 수 있으므로 자동으로 부하 분산과 장애 내성이 생긴다.

```
[클라이언트] → 워크플로우 시작 요청
                    ↓
           [Temporal Server]  ← Event History DB
                    ↓ 태스크 배분 (폴링)
           [Worker (Spring Boot)]
           → PaymentActivity 실행
           → InventoryActivity 실행
           → 완료 보고 → [Temporal Server]
```

---

## 핵심 동작 원리: Event History + Replay

Temporal이 "서버가 죽어도 이어진다"는 걸 어떻게 보장하는가.

Temporal 서버는 Workflow 실행 중 발생한 모든 사건을 **Event History**로 기록한다.

```
[Event History]
1. WorkflowStarted
2. ActivityScheduled(charge)
3. ActivityCompleted(charge) ← 결제 성공
4. ActivityScheduled(deduct)
← 여기서 Worker 죽음
```

Worker가 재시작되면 Temporal은 Workflow 코드를 처음부터 다시 실행하되, Event History를 보면서 **이미 완료된 Activity는 실제로 실행하지 않고 결과만 복원**한다. 결제를 다시 청구하지 않고, 기록된 결과값을 그대로 쓴다.

이 방식을 **Deterministic Replay**라고 한다. Workflow 코드가 결정적이어야 하는 이유가 바로 이 Replay 때문이다.

Worker는 상태를 가지지 않는다. 본인이 기억하는 게 아니라 **Temporal Server에서 Event History를 받아서 현재 상태를 재구성**한다. 상태는 Temporal이 독점 관리, 실행은 Worker가 담당 — 역할이 완전히 분리된 구조다.

---

## Stub: Workflow에서 Activity를 호출하는 방식

Workflow 코드 안에서 Activity를 직접 `new` 해서 호출하면 안 된다. Temporal이 중간에서 실행을 제어해야 하기 때문이다.

그래서 **Stub**을 통해 호출한다.

```java
// 직접 호출 (X) — Temporal이 개입 못함
PaymentActivityImpl payment = new PaymentActivityImpl();
payment.charge(1000);

// Stub을 통한 호출 (O) — Temporal이 실행 제어
PaymentActivity payment = Workflow.newActivityStub(
    PaymentActivity.class,
    ActivityOptions.newBuilder()
        .setStartToCloseTimeout(Duration.ofSeconds(10))
        .setRetryOptions(RetryOptions.newBuilder()
            .setMaximumAttempts(3)
            .build())
        .build()
);
payment.charge(1000); // 실제론 Temporal Server에 태스크로 등록됨
```

`stub.charge()`를 호출하는 순간 실제로 바로 실행되는 게 아니다. Temporal Server에 "charge Activity 실행해줘" 라는 이벤트가 등록되고, Worker가 그걸 폴링해서 실제로 실행한다.

```
Workflow 코드
  → stub.charge() 호출
       ↓
  [Temporal Server] → ActivityScheduled 이벤트 저장
       ↓ (Worker 폴링)
  [Worker] → 실제 charge() 실행
       ↓
  [Temporal Server] → ActivityCompleted 이벤트 저장
       ↓
  Workflow 다음 줄로 진행
```

재시도 횟수, 타임아웃 같은 정책도 Stub 만들 때 같이 설정한다. Stub은 **"Temporal을 통해서 실행하겠다"는 선언**이다.

---

## 도입 시 코드 변경 범위

기존 비즈니스 로직은 거의 안 건드려도 된다. 달라지는 건 "어떻게 호출하냐"는 레이어다.

```java
// 기존 Spring Service — 그대로 유지
@Service
public class PaymentService {
    public void charge(long amount) { ... }
}

// Activity — 기존 Service를 감싸는 얇은 래퍼 (추가)
public class PaymentActivityImpl implements PaymentActivity {
    private final PaymentService paymentService;

    public void charge(long amount) {
        paymentService.charge(amount); // 기존 로직에 그냥 위임
    }
}

// Workflow — 흐름만 선언 (추가)
public class OrderWorkflowImpl implements OrderWorkflow {
    public void processOrder(Order order) {
        payment.charge(order.getAmount());
        inventory.deduct(order.getItemId());
    }
}
```

가장 큰 변화는 기존에 `OrderService.processOrder()` 안에 순차적으로 썼던 흐름을 Workflow로 꺼내는 것이다. 핵심 비즈니스 로직을 건드리는 게 아니라, 그 로직들을 *누가 어떤 순서로 호출하는가*를 Temporal에게 위임하는 구조다.

---

## Saga 패턴과의 비교

Saga 패턴을 직접 구현하면 보상 트랜잭션 로직을 개발자가 모두 짜야 한다.

```java
// Saga 직접 구현 예시 — 보상 로직이 비즈니스 로직만큼 많아짐
try {
    payment.charge();
    try {
        inventory.deduct();
    } catch (Exception e) {
        payment.refund(); // 보상 트랜잭션 직접 호출
        throw e;
    }
} catch (Exception e) {
    // 재시도 로직, 상태 저장, 알림...
}
```

Temporal로 같은 걸 구현하면:

```java
// Temporal — 보상 로직을 Workflow 레벨에서 선언적으로
public void processOrder(Order order) {
    String paymentId = payment.charge(order.getAmount());
    Workflow.registerCompensation(() -> payment.refund(paymentId));

    inventory.deduct(order.getItemId());
    // 실패 시 등록된 compensation이 자동으로 실행됨
}
```

재시도, 타임아웃, 상태 저장, 보상 실행 순서를 Temporal이 담당한다.

---

## Signal과 Query

장기 실행 Workflow에 외부에서 개입할 수 있는 두 가지 방법이 있다.

**Signal**: Workflow에 이벤트를 보낸다. Workflow 상태를 변경할 수 있다.
```java
// 주문 취소 요청
workflowStub.signal("cancel");
```

**Query**: Workflow의 현재 상태를 읽는다. 상태를 변경하지 않는다.
```java
// 현재 진행 단계 조회
String status = workflowStub.query("getStatus", String.class);
```

Signal은 Event History에 기록되어 내구성이 보장되고, Query는 기록되지 않는다.

---

## Spring Boot 연동

```java
@Configuration
public class TemporalConfig {

    @Bean
    public WorkflowClient workflowClient() {
        WorkflowServiceStubs service = WorkflowServiceStubs.newLocalServiceStubs();
        return WorkflowClient.newInstance(service);
    }

    @Bean
    public WorkerFactory workerFactory(WorkflowClient client) {
        WorkerFactory factory = WorkerFactory.newInstance(client);
        Worker worker = factory.newWorker("order-task-queue");
        worker.registerWorkflowImplementationTypes(OrderWorkflowImpl.class);
        worker.registerActivitiesImplementations(new PaymentActivityImpl());
        factory.start();
        return factory;
    }
}
```

Spring의 DI와 자연스럽게 연결된다. Activity 구현체에 `@Service`, `@Repository` 같은 Spring Bean을 주입할 수 있다.

---

## 언제 쓰면 좋은가

- 여러 서비스를 거치는 장기 실행 비즈니스 플로우 (주문, 정산, 온보딩)
- 중간에 외부 시스템 응답을 기다려야 하는 경우 (결제 콜백, 수동 승인)
- 실패 시 부분 롤백이 필요한 분산 트랜잭션
- 재시도 로직, 타임아웃, 상태 관리를 직접 구현하기 복잡한 경우

반대로 단순한 API 호출이나 짧은 요청-응답 흐름에는 과한 도구다.

---

## 요약

- Workflow (흐름 정의) + Activity (실제 작업) + Worker (실행 프로세스) 세 축으로 구성
- Event History + Deterministic Replay로 장애 후 자동 재개 보장
- Workflow 코드는 반드시 결정적이어야 함 (비결정적 코드는 Activity 안으로)
- Saga 패턴의 보상 트랜잭션 로직을 인프라 레벨에서 흡수
- Spring Boot와 자연스럽게 통합, Java SDK 공식 지원
