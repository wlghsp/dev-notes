# Art of Multiprocessor Programming Glossary 진도표
참고: The Art of Multiprocessor Programming, Revised First Edition — Herlihy & Shavit (Morgan Kaufmann, 2012)

전략: 동시성의 이론적 토대를 수학적 직관 없이 "왜 이 개념이 필요한가"로 접근.
구현보다 개념의 의미를 먼저 체화한다.

---

## Part 0 — 동시성 기초 (Ch.1: Introduction)

모든 동시성 논의의 출발점. 이 개념들이 없으면 이후 내용이 공허하다.

- [x] concurrency-why.md — 왜 멀티스레드 프로그래밍이 필요한가. Amdahl's Law까지 포함해서 정리
- [x] shared-object.md — 여러 스레드가 동시에 접근하는 객체. Counter 예시로 본 동시 접근 문제
- [x] mutual-exclusion-problem.md — Alice&Bob 우화. 임계구역에 한 번에 하나의 스레드만 진입해야 하는 이유
- [x] producer-consumer-problem.md — 생산자-소비자 문제. 버퍼를 사이에 둔 조율 문제
- [x] readers-writers-problem.md — 읽기/쓰기 충돌. 대기 없는 해법이 가능한 유일한 경우
- [x] parallelization-limits.md — concurrency-why.md에 Amdahl's Law로 통합, 별도 파일 생성하지 않음

---

## Part I-A — 상호 배제 원리 (Ch.2: Mutual Exclusion)

락 없이 소프트웨어만으로 상호 배제를 달성하려면 무엇이 필요한가.

- [ ] mutual-exclusion-properties.md — safety(mutual exclusion) / liveness(deadlock-free / starvation-free) 두 가지 요건
- [ ] peterson-lock.md — 2개 스레드 상호 배제의 고전적 소프트웨어 해법. turn + flag 두 변수의 역할
- [ ] filter-lock.md — Peterson Lock을 N개 스레드로 확장한 Filter Lock
- [ ] fairness-in-lock.md — 공정성이란 무엇인가. 굶주림(starvation) 없는 락의 조건
- [ ] bakery-algorithm.md — Lamport의 빵집 알고리즘. 번호표 기반 공정 상호 배제

---

## Part I-B — 동시 객체의 정확성 (Ch.3: Concurrent Objects)

"동시에 올바르게 동작한다"는 것을 어떻게 정의하는가.

- [ ] sequential-consistency.md — 실행 결과가 어떤 순차 실행의 결과와 같아야 한다는 조건
- [ ] linearizability.md — 각 연산이 호출~반환 사이 어느 한 시점에 원자적으로 일어난 것처럼 보여야 한다
- [ ] linearization-point.md — 연산이 "원자적으로 일어난 것으로 볼 수 있는" 단일 시점
- [ ] progress-conditions.md — wait-free / lock-free / obstruction-free 세 가지 진행 보장의 차이
- [ ] java-memory-model-basics.md — JMM이란. happens-before 관계. volatile과 synchronized의 보장

---

## Part I-C — 공유 메모리와 원자적 연산 (Ch.4~6)

하드웨어 수준에서 동시성을 지원하는 원시 연산들.

- [ ] atomic-register.md — 원자적 레지스터란. SRSW / MRSW / MRMW 분류
- [ ] cas.md — Compare-And-Set(CAS). 현대 lock-free 알고리즘의 핵심 원시 연산
- [ ] consensus-number.md — 동기화 프리미티브의 강도를 측정하는 이론적 척도
- [ ] consensus-problem.md — 여러 스레드가 하나의 값에 합의하는 문제. FLP 불가능성과의 관계
- [ ] read-modify-write.md — getAndSet / getAndAdd / compareAndSet 등 RMW 연산 분류

---

## Part II-A — 실용 락 구현 (Ch.7~8)

실제 하드웨어에서 락을 어떻게 효율적으로 구현하는가.

- [ ] test-and-set-lock.md — TAS 락. 가장 단순한 스핀 락. 캐시 경합 문제
- [ ] test-and-test-and-set-lock.md — TTAS 락. 로컬 캐시를 먼저 읽어 캐시 트래픽 감소
- [ ] backoff-lock.md — 경합 시 재시도 전 대기. exponential backoff
- [ ] queue-lock.md — 스레드를 큐에 세워 공정하게 락을 전달. CLH / MCS 락
- [ ] monitor.md — Java의 synchronized가 내부적으로 사용하는 모니터 구조
- [ ] reentrant-lock.md — 같은 스레드가 재진입 가능한 락. ReentrantLock의 내부 동작
- [ ] readers-writers-lock.md — 읽기 동시 허용, 쓰기 독점. Java ReadWriteLock 내부 구조

---

## Part II-B — Lock-Free 자료구조 (Ch.9~11)

락 없이 동시성을 달성하는 자료구조 설계.

- [ ] lock-free-concept.md — lock-free가 무엇인가. 왜 락보다 나을 수 있는가. ABA 문제 소개
- [ ] lock-free-stack.md — CAS 기반 lock-free 스택 구현 원리
- [ ] lock-free-queue.md — Michael-Scott Queue. lock-free 큐의 표준 구현
- [ ] aba-problem.md — ABA 문제란. CAS의 숨겨진 함정. stamped reference로 해결

---

## Part II-C — 동시 자료구조 (Ch.12~14)

실용적인 동시 자료구조 설계 패턴.

- [ ] concurrent-list-locking-strategies.md — coarse-grained / fine-grained / optimistic / lazy 동기화 전략 비교
- [ ] concurrent-hashmap.md — 동시 해시맵 구현. striped locking과 lock-free 해시셋
- [ ] concurrent-skiplist.md — 동시 스킵리스트. ConcurrentSkipListMap의 내부 원리

---

진행 방식: Part 0부터 순서대로. 완료된 항목은 [x]로 표시.
Part I은 이론 무게가 있으니 하나씩 천천히. Part II부터 구현 감각이 붙는다.
