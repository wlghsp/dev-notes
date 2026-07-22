# Externalized Event

Spring Modulith에서 `@Externalized` 어노테이션을 붙인 이벤트가 애플리케이션 내부(In-Process)와 외부 메시지 브로커(Kafka 등) 양쪽으로 동시에 발행되게 하는 기능이다.
`spring-modulith-events-kafka` 같은 의존성을 추가하면 사용할 수 있다.

## 사용 방식

```kotlin
@Externalized("room-rate-changed::#{roomRateId}")
data class RoomRateChangedEvent(val roomRateId: Long)
```

이렇게 선언하면 이 이벤트가 발행될 때 같은 애플리케이션 안의 `@ApplicationModuleListener` 구독자에게도 전달되고, 동시에 지정한 Kafka 토픽으로도 발행된다.

## 왜 필요한가

모듈러 모놀리스로 시작했지만, 특정 모듈은 나중에 정말로 별도 서비스로 분리될 가능성이 있는 경우가 있다.
예를 들어 여러 모듈의 변경을 구독해서 검색 데이터를 동기화하는 검색 모듈은, 트래픽이 커지면 별도 서비스로 떼어낼 후보가 되기 쉽다.
이 모듈을 분리하는 시점이 왔을 때, In-Process 이벤트 발행 코드를 통째로 메시지 브로커 발행 코드로 다시 짜야 한다면 전환 비용이 크다.
`@Externalized`를 미리 붙여두면 발행 로직은 그대로 두고 구독 측만 새 서비스로 옮기면 되므로, 모듈 분리 시점의 작업이 훨씬 가벼워진다.

## Spring Modulith의 MSA 전환 전략에서의 위치

이 기능은 spring-modulith.md에서 다룬 "이벤트로 짠 코드는 브로커로 바꾸는 선에서 전환이 끝난다"는 설명을 실제로 구현하는 구체적인 메커니즘이다.
모듈 경계가 코드 레벨에서 이미 명확히 정의돼 있고 이벤트도 이미 외부 발행 형태를 갖추고 있다면, "어디를 잘라서 분리할 것인가"에 대한 답이 설계 단계가 아니라 이미 코드에 존재하는 셈이 된다.
