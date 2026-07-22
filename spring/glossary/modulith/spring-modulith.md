# Spring Modulith

하나의 Spring Boot 애플리케이션(단일 배포 단위) 안에서 논리적인 모듈 경계를 강제하고 검증할 수 있게 해주는 스프링 프로젝트다.
"모놀리스는 무조건 나쁘다"는 흐름에 대한 반작용으로, 잘 구조화된 모놀리스인 모듈러 모놀리스를 정식으로 지원하려는 목적에서 나왔다.

## 왜 필요한가

Spring Boot로 모놀리스를 짤 때 패키지는 `com.example.order`, `com.example.inventory`처럼 나눠도, 자바 컴파일러는 그 경계를 신경 쓰지 않는다.
`order` 패키지의 내부 클래스를 `inventory` 패키지에서 그냥 import해서 쓸 수 있고, 이런 일이 쌓이면 패키지 구분은 이름만 남고 실질적으로는 전부 뒤엉킨 빅 볼 오브 머드(big-ball-of-mud.md 참고)가 된다.
Spring Modulith는 이 문제를 나중에 MSA로 쪼갤 때가 아니라 지금, 컴파일 타임과 테스트 타임에 잡아준다.

## 모듈 인식 방식

Spring Modulith는 애플리케이션의 최상위 패키지 바로 아래에 있는 각 패키지를 하나의 모듈로 인식한다.
예를 들어 `com.example.app` 아래 `order`, `inventory`, `shipping` 패키지가 있으면 이 셋이 각각 독립된 모듈이 된다.

각 모듈에서 외부에 공개할 것은 모듈 최상위 패키지에 두고, 숨기고 싶은 구현 세부사항은 `internal`이라는 하위 패키지에 넣는다.
`order.internal.OrderRepository`는 다른 모듈에서 접근할 수 없어야 하는 클래스다.
이 규칙을 어기고 `inventory` 모듈이 `order.internal` 패키지의 클래스를 직접 import하면, `ApplicationModules.verify()`라는 테스트 API가 이를 감지해서 테스트를 실패시킨다.
즉 아키텍처 규칙 위반이 코드 리뷰가 아니라 CI 단계에서 걸러진다.

## 모듈 간 통신

모듈 간 통신 방법은 두 가지가 있고, Spring Modulith는 둘 다 허용한다.

첫 번째는 직접 호출이다. `order` 모듈이 `inventory` 모듈의 공개 인터페이스(최상위 패키지의 클래스)를 그냥 메소드 호출로 부르는 방식이다.
`internal` 패키지만 침범하지 않으면 이 방식도 `ApplicationModules.verify()` 검증을 통과한다.
다만 `order`가 `inventory`의 메소드 시그니처와 반환 타입에 컴파일 타임으로 묶이기 때문에, `inventory`의 API가 바뀌면 `order` 코드도 같이 고쳐야 한다.

두 번째는 스프링 이벤트(`ApplicationEventPublisher`)를 쓰는 방식이다.
예를 들어 `order` 모듈이 주문 완료 이벤트를 발행하면 `inventory` 모듈이 그 이벤트를 구독해서 재고를 차감한다.
발행자인 `order`는 누가 이 이벤트를 구독하는지, 심지어 구독자가 있는지조차 몰라도 된다.
이런 통신 방식을 이벤트 기반 아키텍처(Event-Driven Architecture, EDA)라고 부른다.

두 방식의 트레이드오프는 결합도와 즉시성의 맞교환이다.
직접 호출은 결과를 바로 돌려받을 수 있어서 조회처럼 즉시 응답이 필요한 흐름에 맞고, 이벤트는 결합이 느슨한 대신 비동기라 응답을 바로 받지 못한다.
그래서 실무에서는 조회는 직접 호출로, 트랜잭션 결과에 따라 다른 모듈이 반응해야 하는 흐름(주문 완료 → 재고 차감, 알림 발송)은 이벤트로 처리하는 식으로 섞어 쓰는 경우가 많다.

이벤트 방식을 썼을 때 MSA 전환 비용도 더 낮아진다.
직접 호출로 짠 코드는 서비스가 분리되면 메소드 호출을 REST나 gRPC 같은 원격 호출로 다시 짜야 한다.
이벤트로 짠 코드는 `ApplicationEventPublisher`를 Kafka 같은 메시지 브로커로 바꾸는 선에서 끝나는 경우가 많고, 발행·구독 로직 자체는 거의 바뀌지 않는다.

다만 모든 통신을 이벤트로 처리할 수 있는 건 아니다. 어떤 처리가 이벤트에 적합한지 판단하는 기준은 event-driven-suitability.md 참고.

## 예시

```
com.example.shop
├── order
│   ├── OrderController.java       (공개 API)
│   ├── OrderCreatedEvent.java     (공개 이벤트)
│   └── internal
│       ├── OrderRepository.java   (모듈 내부 전용)
│       └── OrderEntity.java
└── inventory
    ├── InventoryController.java
    └── internal
        └── StockService.java
```

테스트 코드로 이 구조가 실제로 지켜지는지 검증할 수 있다.

```java
class ModularityTests {
    ApplicationModules modules = ApplicationModules.of(ShopApplication.class);

    @Test
    void verifiesModularStructure() {
        modules.verify();  // internal 패키지 침범, 순환 참조 등을 검사
    }

    @Test
    void writeDocumentation() {
        new Documenter(modules).writeDocumentation();  // 모듈 구조를 다이어그램으로 자동 생성
    }
}
```

`writeDocumentation()`을 실행하면 모듈 간 관계를 PlantUML, C4 다이어그램으로 자동 생성해준다.
아키텍처 문서가 코드와 따로 노는 문제를 어느 정도 줄여준다.

## 정리

Spring Modulith는 새로운 프레임워크가 아니라, 이미 짜고 있는 Spring Boot 모놀리스에 모듈 경계를 강제하는 검증 도구에 가깝다.
MSA로 가기 전 단계로 모듈러 모놀리스를 실험해보고 싶거나, 이미 커진 모놀리스의 패키지 뒤엉킴을 CI에서 잡고 싶을 때 유용하다.
