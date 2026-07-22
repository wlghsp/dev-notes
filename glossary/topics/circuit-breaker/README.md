# 학습 순서 — 서킷브레이커와 파생 개념

이 폴더는 두 성격의 문서를 분리해서 담는다.

- basics/ — 언제 어디서나 통하는 원초적 개념. 특정 프레임워크나 프로젝트에 종속되지 않는다.
- spring-practice/ — Spring·Resilience4j·Feign이라는 구체적 스택에서 실제로 적용할 때 걸리는 함정과 실무 팁.

basics를 먼저 다 이해한 뒤에 spring-practice로 넘어가는 순서를 권장한다. spring-practice의 많은 파일이 "이 basics 개념이 실전에서는 이렇게 어긋난다"는 구조로 쓰여 있어서, 원형을 모르면 무엇이 어긋난 건지 알아채기 어렵다.

## basics — 원초적 개념

### 0단계 — 서킷브레이커보다 먼저 있던 개념

1. timeout.md — 가장 먼저 읽는다. "왜 무한정 기다리면 안 되는가"라는, 장애 대응 전체의 출발점이 되는 질문을 여기서 잡는다
2. retry.md — 타임아웃으로 실패를 확정한 다음 자연스럽게 나오는 다음 질문("그럼 다시 시도하면 안 되나?")에 답한다. "무한정 재시도하면 안 된다"는 문제의식이 서킷브레이커로 이어진다

### 1단계 — 서킷브레이커란 무엇인가

3. circuit-breaker.md — 이름의 유래(전기 회로 차단기), 타임아웃·재시도만으로는 왜 부족한지, 무엇을 해결하는 패턴인지를 여기서 잡는다. 이후 모든 파일이 이 개념을 전제로 한다
4. circuit-breaker-state-transition.md — CLOSED/OPEN/HALF_OPEN 세 상태와, "CLOSED 상태에서도 실패는 그대로 겪는다"는 핵심 한계를 잡는다. 이 한계가 뒤의 여러 파일에서 반복 등장한다
5. circuit-breaker-fail-fast.md — 서킷브레이커가 OPEN 상태에서 정확히 무엇을 하는지 한 개념으로 짚는다. state-transition.md의 OPEN 상태를 더 깊게 본다

### 2단계 — 서킷브레이커와 나란한 패턴들

6. bulkhead.md — 서킷브레이커와 자주 같이 언급되는 자원 격리 패턴. 이름의 유래(배의 격벽)와 판단 축(시간 vs 자원)의 차이를 잡는다
7. rate-limiter.md — Bulkhead·서킷브레이커와 나란히 놓고 셋의 판단 기준(실패율 vs 자원 사용량 vs 호출 횟수)을 비교한다
8. fallback.md — 서킷브레이커·Bulkhead가 개입했을 때 실제로 무엇을 반환하는지를 다룬다. 언제 실행되는지, 무엇을 반환할지가 왜 설계 선택인지를 짚는다
9. graceful-degradation.md — fallback이 실제로 하는 일의 상위 철학이다. "실패하면 죽는 대신 기능을 낮춰서라도 응답한다"는 개념으로 fallback.md를 마무리한다

### 3단계 — 배경이 되는 세부 개념

10. connect-timeout-vs-read-timeout.md — timeout.md에서 다룬 개념을 "연결 자체가 안 됨"과 "응답이 없음" 두 종류로 세분화한다
11. spring-aop-proxy.md — Spring에서 `@CircuitBreaker` 같은 어노테이션이 실제로 어떻게 동작하는지(프록시 기반)를 다룬다. spring-practice의 여러 파일이 이 개념을 전제로 한다
12. resilience4j.md — 지금까지의 개념들(CircuitBreaker, Retry, Bulkhead, RateLimiter)을 실제로 구현해서 제공하는 Java 라이브러리를 소개한다. Spring 환경에서의 기본 사용법을 짚고 spring-practice로 넘어갈 다리를 놓는다
13. wiremock.md — 이 개념들을 로컬에서 실습·검증할 때 쓰는 mock 서버 도구를 소개한다

## spring-practice — Spring/Resilience4j 적용 실전

basics를 읽은 뒤 이 순서로 읽는다. 각 파일은 "basics의 어떤 개념이 실전에서 어떻게 걸리는가"를 다룬다.

### 4단계 — Spring에서 적용할 때 걸리는 함정

14. self-invocation-and-spring-aop-proxy.md — spring-aop-proxy.md에서 다룬 프록시 메커니즘이 self-invocation 상황에서 왜 무력화되는지를 다룬다
15. feign-proxy-and-aop-annotation-limit.md — self-invocation과는 다른 원인으로 같은 증상(어노테이션 무시)이 나는 경우다. Feign 인터페이스 자체의 프록시 구조 때문이라는 걸 14번과 비교하며 이해한다
16. resilience4j-fallback-method-contract.md — fallback.md에서 다룬 개념을 실제 Resilience4j 코드로 지정할 때 지켜야 할 시그니처 규칙과, 어겼을 때 컴파일은 되지만 런타임에야 드러나는 이유를 다룬다

### 5단계 — 서킷브레이커만으로는 안 끝나는 부분

17. connection-pool-timeout-vs-circuit-breaker.md — "타임아웃 설정만 줄이면 되지 않나?"라는 질문에 답한다. connect-timeout-vs-read-timeout.md와 circuit-breaker-state-transition.md의 한계를 연결해서 이해한다
18. scheduled-fixed-delay-and-blocking-propagation.md — circuit-breaker-state-transition.md의 CLOSED 구간 한계가 Spring 스케줄러 환경에서 실제로 어떻게 나타나는지 실측 타임라인으로 본다
19. task-scheduler-vs-async-executor.md — 18번 문제의 해결책이다. `@Async`로는 왜 안 되는지, 전용 `TaskScheduler`로는 왜 되는지를 비교한다
20. bulkhead-vs-circuit-breaker.md — bulkhead.md에서 다룬 개념이 서킷브레이커와 정확히 어떻게 역할을 나누는지, Spring 환경의 구체적 상황으로 정리한다

### 6단계 — 관측·검증

21. resilience4j-actuator-circuitbreaker-events.md — circuit-breaker-state-transition.md와 circuit-breaker-fail-fast.md에서 다룬 상태 전이가 실제로 로그가 아니라 Actuator로 어떻게 정확히 관측되는지를 다룬다

### 7단계 — 실습 환경에서 자주 걸리는 것

22. wiremock-mapping-reload.md — wiremock.md에서 소개한 도구를 실제로 쓸 때 자주 걸리는 함정이다. 매핑 파일을 고쳐도 컨테이너를 재시작해야 반영된다는 사실
23. rfc5737-test-net.md — connect timeout(10번)을 재현할 때 쓰는 표준 테스트 IP 대역이다. 10번을 읽은 뒤에 봐야 왜 필요한지 이해된다

## 읽고 나서 확인할 것

"타임아웃이 있는데 왜 재시도가 필요하고, 재시도가 있는데 왜 서킷브레이커가 또 필요한가"를 basics만으로 한 문장씩 설명할 수 있는지 점검해본다. 그다음 "서킷브레이커를 붙였는데 왜 안 되지?"라는 질문에는 원인이 self-invocation인지 Feign 프록시 문제인지 fallback 시그니처 문제인지 spring-practice 파일로 구분해서 답할 수 있는지, "서킷브레이커만 붙이면 다 해결되나?"라는 질문에는 CLOSED 구간의 한계와 Bulkhead·Rate Limiter의 필요성으로 답할 수 있는지 확인한다. basics는 원리를, spring-practice는 그 원리가 실전에서 어디서 어긋나는지를 답할 수 있으면 이 폴더는 다 이해한 것이다.
