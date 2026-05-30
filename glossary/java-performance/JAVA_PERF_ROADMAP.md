# Java Performance Glossary 진도표
참고: Java Performance: The Definitive Guide — Scott Oaks (O'Reilly, 2014)

전략: JVM이 "왜 이렇게 동작하는가"를 이해하는 것이 목표.
튜닝 파라미터 암기가 아니라, 내부 동작 원리를 체화하는 방향으로 진행한다.

---

## Phase 0 — JVM 기초 (선행 필수)

JVM을 처음 접하는 사람이 가장 먼저 잡아야 할 개념들.
이것 없이는 GC도, JIT도 이해하기 어렵다.

- [x] jvm-overview.md — JVM이 무엇인가. 왜 존재하는가. 프로세스로서의 JVM
- [x] bytecode.md — .class 파일이란 무엇인가. 소스코드와 실행 사이의 중간 표현
- [x] classloading.md — 클래스가 메모리에 올라오는 과정. bootstrap / extension / app classloader
- [x] jvm-memory-areas.md — heap / stack / method area / PC register / native method stack 각 영역의 역할
- [x] stack-vs-heap.md — 메서드 호출 시 스택 프레임 생성, 객체는 힙에. 둘의 수명 차이
- [x] object-creation.md — new 키워드 호출 시 JVM 내부에서 일어나는 일. TLAB 개념 포함
- [x] gc-why.md — 왜 GC가 필요한가. C/C++와의 차이. GC가 해결하는 문제

---

## Phase 1 — JIT 컴파일러 (Ch.4)

JVM이 바이트코드를 어떻게 실행하고 최적화하는가.

- [ ] interpreted-vs-compiled.md — 인터프리터 방식 vs JIT 컴파일 방식의 차이
- [ ] jit-compiler.md — Just-In-Time 컴파일이란. 왜 처음부터 컴파일하지 않는가
- [ ] hotspot-compilation.md — 자주 실행되는 코드(hot code)를 감지해 컴파일하는 원리
- [ ] tiered-compilation.md — C1(client) / C2(server) 컴파일러의 역할과 5단계 컴파일 레벨
- [ ] code-cache.md — JIT 컴파일된 코드가 저장되는 메모리 영역. 가득 차면 어떻게 되는가
- [ ] inlining.md — 메서드 호출을 호출 지점에 직접 삽입하는 최적화. 가장 중요한 JIT 최적화
- [ ] escape-analysis.md — 객체가 메서드 밖으로 탈출하지 않으면 힙 할당을 생략하는 최적화
- [ ] deoptimization.md — 컴파일된 코드가 다시 인터프리터로 돌아가는 경우와 이유

---

## Phase 2 — GC 기초 (Ch.5)

GC가 무엇을 하고, 왜 stop-the-world가 발생하는가.

- [x] gc-overview.md — GC의 기본 역할. mark / sweep / compact 세 단계
- [ ] generational-gc.md — 왜 세대(generation)로 나누는가. weak generational hypothesis
- [ ] young-generation.md — Eden / Survivor 0 / Survivor 1 구조. minor GC 동작 흐름
- [ ] old-generation.md — 오래 살아남은 객체가 이동하는 공간. major GC 발생 조건
- [ ] stop-the-world.md — GC 중 애플리케이션 스레드가 멈추는 현상. 왜 발생하는가
- [ ] gc-roots.md — GC가 살아있는 객체를 판단하는 시작점. stack / static field / JNI references
- [ ] object-promotion.md — 객체가 young → old로 이동하는 조건. tenuring threshold

---

## Phase 3 — GC 알고리즘 (Ch.6)

각 GC 알고리즘이 어떻게 다르게 동작하는가.

- [ ] serial-gc.md — 단일 스레드 GC. 언제 쓰는가
- [ ] parallel-gc.md — throughput collector. 여러 스레드로 GC. pause time vs throughput 트레이드오프
- [ ] cms-gc.md — Concurrent Mark Sweep. 애플리케이션과 동시에 mark하는 원리. concurrent mode failure
- [ ] g1-gc.md — Garbage First. region 기반 분할. 예측 가능한 pause time 목표
- [ ] gc-algorithm-tradeoffs.md — throughput / latency / footprint 세 축에서 각 GC의 위치

---

## Phase 4 — 힙 메모리 관리 (Ch.7)

힙을 어떻게 분석하고 문제를 찾는가.

- [ ] heap-analysis.md — heap histogram과 heap dump의 차이. 언제 어떤 것을 쓰는가
- [ ] oom-error.md — OutOfMemoryError의 종류별 원인. heap space / GC overhead / metaspace
- [ ] memory-leak-jvm.md — Java에서 메모리 누수가 발생하는 패턴. 참조가 끊기지 않는 이유
- [ ] string-interning.md — String pool과 intern(). 언제 유용하고 언제 위험한가
- [ ] weak-soft-references.md — WeakReference / SoftReference / PhantomReference 각각의 GC 처리 방식

---

## Phase 5 — 네이티브 메모리 (Ch.8)

힙 바깥에서 JVM이 사용하는 메모리.

- [ ] native-memory.md — JVM이 힙 외에 사용하는 메모리 영역들. metaspace / code cache / thread stack
- [ ] metaspace.md — PermGen이 사라지고 등장한 Metaspace. 클래스 메타데이터 저장 공간
- [ ] compressed-oops.md — 64비트 JVM에서 객체 포인터를 32비트로 압축하는 기법

---

## Phase 6 — 스레딩과 동기화 성능 (Ch.9)

멀티스레드 환경에서 JVM이 동기화를 어떻게 처리하는가.

- [ ] thread-pool.md — ThreadPoolExecutor 구조. core / max 스레드, 큐 크기의 관계
- [ ] synchronized-cost.md — synchronized 키워드가 성능에 미치는 영향. 락 획득 비용
- [ ] lock-biasing.md — Biased Locking. 경합이 없을 때 락 비용을 줄이는 JVM 최적화
- [ ] lock-spinning.md — 락 대기 시 스레드를 sleep 대신 spin하게 하는 최적화
- [ ] false-sharing.md — 다른 변수를 쓰는 스레드들이 같은 캐시 라인을 공유해 성능이 저하되는 현상
- [ ] volatile-jvm.md — volatile 키워드가 JVM과 CPU 레벨에서 하는 일. memory barrier

---

진행 방식: Phase 0부터 순서대로. 각 Phase 완료 후 다음 Phase 이동.
완료된 항목은 [x]로 표시.
