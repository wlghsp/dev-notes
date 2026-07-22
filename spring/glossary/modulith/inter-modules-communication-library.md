# 자체 제작 모듈 간 통신 라이브러리

Spring Modulith 같은 기성 라이브러리를 쓰지 않고, 모듈 간 통신(직접 호출과 이벤트 발행 모두)을 직접 만든 별도 라이브러리로 추상화하는 패턴이다.
당근마켓 스프링캠프 발표(박용권, 2024)에서 `modulemesh`라는 이름으로 소개된 자체 구현 사례가 있다.

## 왜 필요한가

모듈 A가 모듈 B의 인터페이스를 직접 참조하는 방식은 컴파일 타임 의존을 만든다.
이 의존을 완전히 없애고 싶을 때, Spring Modulith의 이벤트 방식(`ApplicationEventPublisher`)을 쓰지 않고도 같은 효과를 자체 라이브러리로 구현할 수 있다.
핵심은 모듈 간 호출을 "타입"이 아니라 "이름(문자열 키)"으로 하는 것이다.

## 동기 방식 구현

```
registry.registerModuleFunction("customers/get-customer-details", CustomerDetails.class)
```

`user` 모듈은 이렇게 자신이 제공하는 기능을 문자열 키(`"customers/get-customer-details"`)와 반환 타입으로 레지스트리에 등록해둔다.
`order` 모듈은 이 키를 이용해서 실행을 요청한다.

```
operations.execute("customers/get-customer-details", Orderer.class)
```

중간의 `InterModules Communication Processor`가 이 요청을 받아 등록된 함수를 실행하고, 결과 타입을 호출자가 원하는 타입(`CustomerDetails`에서 `Orderer`로)으로 변환해서 돌려준다.
이 과정에서 `order` 모듈은 `user` 모듈의 실제 클래스나 인터페이스를 한 번도 직접 import하지 않는다 — 오직 문자열 키와 자신의 타입만 안다.

## 비동기(이벤트) 방식 구현

같은 원리를 이벤트 발행에도 적용할 수 있다.

```
eventPublisher.publishEvent(OrderAcceptedEvent.of(orderId))
```

`order` 모듈이 이벤트를 발행하면, `modulemesh` 내부의 `ModuleEventProcessor`가 이 이벤트를 받아 구독 측 모듈(`brew`)이 이해하는 타입으로 변환해서 전달한다.
Spring Modulith의 `@ApplicationModuleListener`가 하는 일과 목적은 같지만, 이 방식에서는 이벤트 타입 변환까지 직접 만든 계층이 담당한다는 차이가 있다.

## Spring Modulith와의 관계

이 패턴은 Spring Modulith가 라이브러리 차원에서 제공하는 기능(모듈 검증, 이벤트 발행/구독, 이벤트 영속화 등)을 쓰지 않는 프로젝트에서, 같은 문제(모듈 간 결합 최소화)를 직접 해결하려 할 때 나오는 접근이다.
타입 대신 이름으로 통신하는 방식은 결합을 더 느슨하게 만들지만, 그 대가로 컴파일 타임에 잡을 수 있었던 오류(존재하지 않는 키, 타입 불일치)가 런타임 오류로 미뤄진다는 트레이드오프가 있다.
