# Spring Modulith

## 왜 등장했나

Spring Boot로 개발하면 처음엔 편하다. 그런데 서비스가 커지면서 `OrderService`가 `UserRepository`를 직접 호출하고, `PaymentService`가 `InventoryService` 내부 구현에 의존하기 시작한다. 패키지 구조는 있지만 경계는 없다. 공개 클래스면 어디서든 쓸 수 있으니까.

이게 쌓이면 "Big Ball of Mud" — 모든 게 모든 것에 의존하는 덩어리가 된다.

이걸 해결하려면 마이크로서비스로 쪼개면 되지만, 네트워크 비용, 분산 트랜잭션, 운영 복잡도가 생긴다. **Spring Modulith는 그 중간 지점을 노린다. 단일 배포 단위(모놀리스)이면서, 모듈 경계를 코드 레벨에서 강제한다.**

---

## 핵심 개념: Application Module

Spring Modulith에서 모듈은 **패키지 단위**다. 애플리케이션 루트 패키지 바로 아래 있는 최상위 패키지 하나 = 모듈 하나.

```
com.example.shop
  ├── order/         ← Order 모듈
  │   ├── OrderService.java       (public — 모듈 API)
  │   ├── OrderController.java    (public — 모듈 API)
  │   └── internal/
  │       ├── OrderRepository.java   (internal — 외부 접근 불가)
  │       └── OrderValidator.java    (internal)
  ├── payment/       ← Payment 모듈
  └── inventory/     ← Inventory 모듈
```

`internal/` 패키지 안의 클래스는 클래스 자체가 `public`이어도, 다른 모듈에서 접근하면 **Spring Modulith가 테스트 시점에 위반으로 감지한다.**

표준 Spring Boot에서 `public`은 어디서든 접근 가능. Spring Modulith에서 `public`은 **같은 모듈 내, 혹은 명시적으로 노출한 API만 다른 모듈이 접근 가능**하다.

---

## 모듈 경계 검증

```java
@Test
void verifyModularStructure() {
    ApplicationModules.of(ShopApplication.class).verify();
}
```

이 테스트 하나로 모든 모듈 간 의존성 위반을 잡아낸다. 빌드 시점이 아닌 **테스트 시점에 아키텍처를 강제**하는 방식이다.

순환 의존도 감지한다. A → B → A 구조가 생기면 `verify()` 실패.

---

## 모듈 간 통신: 이벤트 기반

모듈이 분리됐다면, 모듈끼리 어떻게 소통해야 할까. 직접 호출하면 다시 결합도가 생긴다.

Spring Modulith의 답은 **Spring Application Events**.

```java
// Order 모듈 내부
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher events;

    @Transactional
    public void placeOrder(Order order) {
        // 주문 처리 로직
        events.publishEvent(new OrderPlaced(order.getId()));
    }
}
```

```java
// Payment 모듈 내부
@ApplicationModuleListener
public void onOrderPlaced(OrderPlaced event) {
    // 결제 처리
}
```

`@ApplicationModuleListener`는 세 가지를 합친 것이다.
- `@Async` — 비동기 실행
- `@Transactional` — 리스너 자체도 트랜잭션 내에서 실행
- `@TransactionalEventListener` — 원본 트랜잭션 커밋 후 실행

즉, `placeOrder()` 트랜잭션이 성공적으로 커밋된 후에야 Payment 모듈의 리스너가 실행된다. 주문 저장 실패 시 결제 이벤트는 발행되지 않는다.

---

## Transactional Outbox 패턴 (자동 제공)

이벤트 기반 통신의 고전적인 문제: **트랜잭션 커밋은 됐는데 이벤트 발행은 실패하면?**

Spring Modulith는 이를 **Event Publication Registry**로 해결한다. 이벤트를 발행할 때 동일한 트랜잭션 내에 이벤트를 DB에도 저장한다. 리스너가 성공적으로 처리하면 완료 표시, 실패하면 재시도 가능한 상태로 남긴다.

별도 구현 없이 `spring-modulith-events-*` 의존성만 추가하면 Outbox 패턴이 자동으로 적용된다.

```
[비즈니스 트랜잭션] --(커밋)--> [이벤트 발행 로그에 기록]
                                       |
                              [리스너 비동기 실행]
                                       |
                         성공: 로그 완료 표시 / 실패: 재시도 대기
```

---

## 모듈 문서 자동 생성

```java
new ApplicationModules(ShopApplication.class).forEach(System.out::println);
```

각 모듈의 공개 API, 의존 관계, 이벤트 발행/수신 현황을 출력한다. Asciidoc 형태로도 추출 가능해서 문서화 자동화에 쓸 수 있다.

---

## 모듈 단위 통합 테스트

```java
@ApplicationModuleTest
class OrderModuleTest {
    // Order 모듈만 로딩. 다른 모듈은 Mock 처리됨
}
```

전체 `@SpringBootTest` 없이 특정 모듈만 격리해서 테스트. 다른 모듈 Bean은 자동으로 Mock이 제공된다.

---

## 마이크로서비스와의 관계

Spring Modulith의 모듈 구조는 나중에 마이크로서비스로 분리할 때 자연스러운 경계가 된다. 모듈 간 통신이 이미 이벤트 기반이라면, 이벤트를 Kafka나 RabbitMQ로 외부화하는 것만으로 분리가 가능하다.

`spring-modulith-events-kafka` 같은 모듈을 추가하면 Application Event를 자동으로 외부 메시지 브로커로 라우팅하는 것도 지원한다.

**모놀리스로 시작 → 도메인 경계 안정화 → 필요한 모듈만 마이크로서비스로 추출** 이 흐름을 코드 수정 없이 가능하게 만드는 것이 핵심 가치다.

---

## 요약

- 단일 배포 단위이면서 패키지 = 모듈 경계를 강제한다
- `internal/` 패키지는 외부 모듈에서 접근 불가 (테스트로 검증)
- 모듈 간 통신은 Application Events + `@ApplicationModuleListener`
- Transactional Outbox는 별도 구현 없이 자동 제공
- 마이크로서비스로의 점진적 전환을 고려한 설계 철학
