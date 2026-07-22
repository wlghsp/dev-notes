# Modulith 학습 순서

이 폴더는 Spring Modulith 라이브러리 자체의 개념과, 모듈러 모놀리스를 설계할 때 필요한 프레임워크 중립적인 개념을 함께 담고 있다.
아래 순서대로 읽으면 왜 모듈러 모놀리스가 필요한지에서 시작해서, Spring Modulith로 어떻게 구현하는지, 더 나아가 라이브러리 없이도 같은 원칙을 어떻게 적용하는지까지 이어진다.

## 1단계 — 왜 모듈러 모놀리스인가

big-ball-of-mud.md (상위 폴더 spring/glossary/에 위치) — 모듈 경계가 코드로 강제되지 않으면 결국 어떤 상태가 되는지부터 짚는다.
spring-modulith.md — Spring Modulith가 이 문제를 어떻게 푸는지 전체 그림을 잡는다. 모듈 인식 방식, `verify()` 검증, 모듈 간 통신(직접 호출/이벤트) 두 방식을 여기서 개괄한다.

## 2단계 — 모듈 경계를 얼마나 엄격하게 그을 것인가

application-module-type.md — CLOSED와 OPEN 두 캡슐화 수준의 차이와, 도입 초기엔 OPEN에서 시작해 점진적으로 좁혀가는 전략을 다룬다.
named-interface.md — CLOSED 상태에서 특정 패키지만 선택적으로 공개하는 `@NamedInterface`를 다룬다.

## 3단계 — 모듈 간 통신, 언제 무엇을 쓸 것인가

event-driven-suitability.md — 직접 호출과 이벤트 중 무엇을 쓸지 가르는 판단 기준(즉시성, 트랜잭션 일관성, 순서 보장)을 다룬다. spring-modulith.md의 통신 문단을 먼저 읽은 뒤 보면 이해가 빠르다.
event-publication-registry.md — 이벤트 방식을 선택했을 때, 이벤트 유실을 감지하고 복구하는 안전망(`event_publication` 테이블)을 다룬다.
externalized-event.md — 특정 모듈을 나중에 별도 서비스로 분리할 가능성이 있을 때, 이벤트를 In-Process와 Kafka 양쪽에 동시에 발행하는 `@Externalized`를 다룬다.

## 4단계 — 실무에서 부딪히는 문제

bean-name-collision.md — 여러 모듈이 하나의 Spring Context를 공유하면서 생기는 Bean 이름 충돌과 해결 방식을 다룬다.

## 5단계 — 설계와 의사결정을 기록하고 구조화하는 법

adr.md — 모듈 경계를 어디에 그을지 같은 설계 결정을 기록하는 방법(ADR)을 다룬다. 이후 단계들은 실제로 이런 결정이 필요한 지점들이다.
context-mapping.md — 모듈(바운디드 컨텍스트) 간의 관계를 의도적으로 설계하는 DDD의 컨텍스트 매핑 개념을 다룬다.
database-per-module.md — 모듈 경계를 데이터베이스 계층까지 확장할지, 어떻게 확장할지를 다룬다.

## 6단계 — Spring Modulith 없이 같은 원칙 구현하기 (심화)

여기서부터는 Spring Modulith 라이브러리를 쓰지 않고, 같은 문제를 다른 도구로 푸는 방식들이다. Spring Modulith가 자동으로 해주는 것들을 수동으로 구현하면 무엇을 하는 셈인지 이해하는 데 도움이 된다.

anti-corruption-layer.md — 모듈 도메인 계층이 다른 모듈의 모델에 오염되지 않도록 막는 DDD 패턴. named-interface.md와 반대 방향(밖의 것을 안으로 들일 때)의 문제를 다룬다.
java-platform-module-system.md — 테스트 타임 검증(`verify()`)이 아니라 컴파일 타임에 강제하는 언어 자체의 모듈 시스템(JPMS).
ioc-container-per-module.md — 모듈 경계를 Spring IoC 컨테이너 자체를 나누는 수준까지 강제하는 극단적인 격리 방식.
inter-modules-communication-library.md — Spring Modulith의 이벤트/직접 호출을 대신하는, 이름(문자열 키) 기반의 자체 제작 통신 라이브러리 패턴.

## 7단계 — 테스트 전략

test-pyramid-for-modular-monolith.md — 단위/통합/시스템 테스트를 모듈 경계에 맞춰 어떻게 나누는지 다룬다. 마지막에 두는 이유는, 앞선 모든 설계 결정(모듈 경계, 통신 방식, 데이터 분리)이 실제로 지켜지는지 검증하는 단계이기 때문이다.
