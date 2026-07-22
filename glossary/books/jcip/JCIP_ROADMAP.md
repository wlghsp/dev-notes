# Java Concurrency in Practice Glossary 진도표

참고: Java Concurrency in Practice — Brian Goetz 외 (Addison-Wesley, 2006)

전략: "왜 스레드가 안전하지 않은가"에서 시작해서 "어떻게 안전하게 합성하는가"로 올라가는 흐름을 잡는다.
Part I(기초) → Part II(구조화) → Part III(성능/테스트) → Part IV(고급, JMM) 순서.

---

## Chapter 1 — Introduction

동시성이 왜 필요하고 왜 위험한지, 이 책 전체의 문제의식을 잡는 챕터.

- [x] thread-safety-intro.md — 스레드 안전성이라는 개념 자체. 왜 단일 스레드 사고방식이 깨지는가
- [x] race-condition.md — 경쟁 조건. 타이밍에 따라 결과가 달라지는 근본 원인
- [x] ch01-introduction.md — 챕터 1 종합 문서

---

## Chapter 2 — Thread Safety

Part I 시작. "스레드 안전하다"의 정의와 그걸 지키는 기본 도구.

- [ ] atomicity.md — 원자성. 복합 연산(check-then-act, read-modify-write)이 왜 깨지는가
- [ ] intrinsic-lock.md — 내재적 락(synchronized). 상호 배제와 재진입성
- [ ] guarding-state-with-locks.md — 락으로 상태를 보호한다는 것의 의미. 락과 변수의 연결 규칙

---

## Chapter 3 — Sharing Objects

락이 원자성만이 아니라 가시성도 보장한다는 것을 다루는 챕터.

- [ ] visibility.md — 가시성. 한 스레드의 쓰기가 다른 스레드에 보이지 않을 수 있다는 문제
- [ ] volatile.md — volatile 변수. 가시성만 보장하고 원자성은 보장하지 않는 이유
- [ ] publication-and-escape.md — 발행과 이스케이프. 객체가 의도치 않게 노출되는 경로
- [ ] thread-confinement.md — 스레드 한정. 공유하지 않아서 안전한 경우
- [ ] immutability.md — 불변성. 상태가 변하지 않으면 동기화가 필요 없는 이유
- [ ] safe-publication.md — 안전한 발행. 올바른 초기화가 다른 스레드에 보이도록 만드는 방법

---

## Chapter 4 — Composing Objects

스레드 안전한 부품으로 스레드 안전한 클래스를 만드는 설계 패턴.

- [ ] java-monitor-pattern.md — 자바 모니터 패턴. 상태와 락을 캡슐화하는 관례
- [ ] delegating-thread-safety.md — 스레드 안전성 위임. 이미 안전한 컴포넌트로 합성할 때의 함정

---

## Chapter 5 — Building Blocks

java.util.concurrent가 제공하는 컬렉션과 동기화 도구.

- [ ] concurrent-collection.md — 동기화 컬렉션과 동시성 컬렉션의 차이. 락 스트라이핑
- [ ] blocking-queue.md — 블로킹 큐와 생산자-소비자 패턴
- [ ] synchronizer.md — 동기화 도구. 래치, 세마포어, 배리어의 역할 차이

---

## Chapter 6 — Task Execution

작업을 스레드에 어떻게 매핑할 것인가.

- [ ] executor-framework.md — Executor 프레임워크. 작업 제출과 실행 정책의 분리
- [ ] exploitable-parallelism.md — 병렬화 가능한 작업을 찾는 기준

---

## Chapter 7 — Cancellation and Shutdown

실행 중인 작업을 안전하게 멈추는 법.

- [ ] task-cancellation.md — 작업 취소. 인터럽트를 통한 협조적 취소
- [ ] interruption.md — 인터럽트. 블로킹 메서드가 취소 신호를 받는 방식

---

## Chapter 8 — Applying Thread Pools

스레드 풀을 실제로 튜닝하고 확장하는 법.

- [ ] thread-pool-sizing.md — 스레드 풀 크기 산정. CPU 바운드 vs I/O 바운드 작업의 차이
- [ ] thread-pool-executor-config.md — ThreadPoolExecutor 설정. 큐 정책과 포화 정책

---

## Chapter 9 — GUI Applications

(스킵 가능 — 백엔드 중심 학습에서는 우선순위 낮음)

---

## Chapter 10 — Avoiding Liveness Hazards

Part III 시작. 안전성 다음은 활동성(liveness).

- [ ] deadlock.md — 데드락. 락 순서 뒤바뀜이 만드는 순환 대기
- [ ] lock-ordering.md — 락 순서. 데드락을 피하는 설계 규칙

---

## Chapter 11 — Performance and Scalability

동시성 코드의 성능을 어떻게 측정하고 개선하는가.

- [ ] amdahls-law.md — 암달의 법칙. 병렬화해도 줄지 않는 순차 구간의 한계
- [ ] lock-contention.md — 락 경합. 경합을 줄이는 전략(범위 축소, 스트라이핑, 락 제거)

---

## Chapter 12 — Testing Concurrent Programs

(필요 시 진행 — 실전 테스트 기법)

---

## Chapter 13 — Explicit Locks

synchronized를 넘어서는 명시적 락.

- [ ] reentrant-lock.md — ReentrantLock. synchronized 대비 얻는 것(타임아웃, 공정성, 인터럽트 가능한 락)
- [ ] read-write-lock.md — 읽기-쓰기 락. 읽기 경합을 허용하는 락 분리

---

## Chapter 14 — Building Custom Synchronizers

동기화 도구를 직접 만들 때 필요한 개념.

- [ ] condition-queue.md — 조건 큐. wait/notify로 상태 의존성을 처리하는 방식
- [ ] abstract-queued-synchronizer.md — AQS. java.util.concurrent 동기화 클래스들의 공통 기반

---

## Chapter 15 — Atomic Variables and Nonblocking Synchronization

락 없이 동시성을 처리하는 하드웨어 기반 접근.

- [ ] compare-and-swap.md — CAS. 락 없이 원자적 갱신을 가능하게 하는 하드웨어 명령
- [ ] nonblocking-algorithm.md — 논블로킹 알고리즘. CAS로 락을 대체하는 방식과 트레이드오프

---

## Chapter 16 — The Java Memory Model

이 책의 이론적 토대. 왜 지금까지의 규칙들이 성립하는지 설명하는 챕터.

- [ ] java-memory-model.md — JVM 메모리 모델. 재정렬을 허용하되 happens-before로 순서를 보장하는 계약
- [ ] happens-before.md — happens-before 관계. 가시성을 보장하는 구체적 규칙들
- [ ] initialization-safety.md — 초기화 안전성. final 필드가 생성자 밖으로 안전하게 발행되는 원리

---

진행 방식: Chapter 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 10 → 11 → 13 → 14 → 15 → 16 순서 권장 (9, 12는 선택).
완료된 항목은 [x]로 표시.
