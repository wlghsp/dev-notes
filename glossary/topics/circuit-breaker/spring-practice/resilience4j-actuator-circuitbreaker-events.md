# Actuator로 서킷브레이커 상태 전이 관측하기

Resilience4j 서킷브레이커의 상태(CLOSED/OPEN/HALF_OPEN)와 전이 이력은 Spring Boot Actuator를 통해 실시간으로 조회할 수 있다. 로그만으로는 놓치기 쉬운 전이 시점을 정확히 확인하는 방법이다.

## 필요한 설정

`spring-boot-starter-actuator` 의존성과, `management.endpoints.web.exposure.include`에 `health`, `circuitbreakers`(엔드포인트 이름의 s 유무는 버전에 따라 다를 수 있다)를 노출해야 한다.

## 조회 방법

현재 서킷브레이커들의 상태 스냅샷을 본다.

```bash
curl http://localhost:<port>/actuator/circuitbreakers
```

특정 서킷브레이커의 이벤트 이력(호출 성공/실패/스킵, 상태 전이)을 시간순으로 본다.

```bash
curl http://localhost:<port>/actuator/circuitbreakerevents/<name>
```

## 확인해야 할 이벤트 종류

`ERROR` 이벤트는 실제로 호출을 시도했다가 실패한 경우다. `NOT_PERMITTED` 이벤트는 서킷이 이미 열려 있어 호출 자체를 막은 경우다. 이 두 이벤트 타입의 전환 지점을 보면, 서킷이 정확히 언제 OPEN으로 전환됐는지 알 수 있다 — `ERROR`가 반복되다가 `NOT_PERMITTED`로 바뀌는 지점이다.

`STATE_TRANSITION` 이벤트는 상태가 실제로 바뀐 순간을 기록한다. `OPEN → HALF_OPEN → CLOSED` 순서로 찍히면 정상적으로 복구된 것이다.

## 왜 로그보다 이 방법이 정확한가

애플리케이션 로그는 개발자가 직접 남긴 지점에서만 정보를 남기기 때문에, "정확히 몇 번째 호출에서 OPEN으로 전환됐는가" 같은 질문에 답하기 어려울 수 있다. Actuator의 circuitbreakerevents는 Resilience4j 내부에서 상태가 바뀔 때마다 자동으로 기록하는 이벤트라 더 정확하고 세밀하다.

참고: circuit-breaker-state-transition.md
참고: circuit-breaker-fail-fast.md
