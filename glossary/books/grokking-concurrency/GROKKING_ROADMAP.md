# Grokking Concurrency Glossary 진도표
참고: Grokking Concurrency (2024, Manning) — Kirill Bobrov

---

## Chapter 1 — Introducing Concurrency

- [x] concurrency.md — 동시성의 정의, 왜 필요한가
- [x] latency.md — 단일 작업 완료 시간
- [x] throughput.md — 단위 시간당 처리량
- [x] moores-law.md — 트랜지스터 2년마다 2배, 멀티코어 전환의 배경
- [x] scalability.md — vertical vs horizontal scaling
- [x] vertical-scaling.md — 서버 사양 업그레이드
- [x] horizontal-scaling.md — 서버 추가로 부하 분산
- [x] decoupling.md — what과 when을 분리하는 전략

## Chapter 2 — Serial and Parallel Execution

- [x] serial-execution.md — 순차 실행 방식
- [x] sequential-computation.md — 이전 결과에 의존하는 연산 순서
- [x] parallel-execution.md — 물리적으로 동시에 실행
- [x] amdahls-law.md — 직렬 구간이 병렬화의 천장을 만든다
- [x] gustafsons-law.md — 코어가 늘면 더 큰 문제를 같은 시간에 풀 수 있다
- [x] concurrency-vs-parallelism.md — 동시성과 병렬성의 차이

## Chapter 3 — How Computers Work

- [x] processor.md — CPU 구조, CU와 ALU 역할
- [x] cache.md — 캐시 3단계(L1/L2/L3), 캐시 컨트롤러, scaled latency
- [x] cpu-execution-cycle.md — Fetch-Decode-Execute-Store 4단계 사이클
- [x] runtime-system.md — 런타임 시스템 개념, 컴퓨터 시스템 구성 요소
- [x] operating-system.md — OS, 시스템 콜, user/kernel space
- [x] instruction-level-parallelism.md — 명령어 수준 병렬성, 동시성 하드웨어 계층에서의 위치
- [x] multiprocessor.md — 멀티프로세서, 멀티코어, SMP, 캐시 일관성(MESI)
- [x] computer-cluster.md — 분산 메모리, loosely/tightly coupled
- [x] flynns-taxonomy.md — SISD/MISD/SIMD/MIMD 분류 체계
- [x] cpu-vs-gpu.md — CPU(MIMD) vs GPU(SIMD) 설계 차이와 적합한 작업 유형

## Chapter 4 — Building Blocks of Concurrency

- [x] process.md — OS가 관리하는 실행 단위, PCB, 프로세스 상태
- [x] thread.md — 프로세스 내 독립적인 실행 흐름, 공유/비공유 자원

## Chapter 5 — Interprocess Communication

- [x] ipc.md — 프로세스 간 통신 방식 개요, 공유 메모리 vs 메시지 패싱
- [x] shared-memory.md — 공유 메모리 기반 IPC, 동기화 책임
- [x] message-passing.md — 메시지 패싱 기반 IPC, 동기/비동기, Erlang/Go
- [x] thread-pool.md — 스레드 풀 패턴, 스레드 수 결정 기준

## Chapter 6 — Multitasking

- [x] multitasking.md — OS 멀티태스킹 동작 방식, preemptive 방식, context switching 비용
- [x] cpu-bound.md — CPU 연산이 병목인 작업, 멀티코어로 성능 향상 가능
- [x] io-bound.md — I/O 대기가 병목인 작업, 코어 추가보다 비동기 방식이 효과적
- [x] scheduler.md — OS 스케줄러 동작 원리, ready queue, time-sharing, 4가지 목표

## Chapter 7 — Decomposition

- [x] task-decomposition.md — 기능별로 독립 작업으로 쪼개는 전략, 의존성 그래프, MIMD/MISD 적합
- [x] data-decomposition.md — 데이터를 청크로 나눠 병렬 처리, map/fork-join/map-reduce 패턴
- [x] pipeline-pattern.md — 단계별 처리 파이프라인, 스레드+큐 구현, 공유 자원 제한 시 유용
- [x] granularity.md — 분해 세분화 정도, fine vs coarse 트레이드오프, agglomeration

## Chapter 8 — Race Conditions and Synchronization

- [ ] race-condition.md — 경쟁 조건, 데이터 레이스
- [ ] shared-resource.md — 동시 접근이 문제가 되는 공유 자원
- [ ] synchronization.md — 동기화 기법 개요
- [ ] mutex.md — 뮤텍스, 임계구역 보호
- [ ] lock.md — 락의 동작 원리

## Chapter 9 — Deadlocks and Starvation

- [ ] deadlock.md — 교착 상태 발생 조건
- [ ] livelock.md — 교착은 아니지만 진행이 안 되는 상태
- [ ] starvation.md — 특정 작업이 계속 실행 기회를 못 얻는 문제

## Chapter 10 — Nonblocking I/O

- [ ] blocking-io.md — I/O 완료까지 블로킹하는 방식
- [ ] nonblocking-io.md — I/O 완료를 기다리지 않고 반환하는 방식

## Chapter 11 — Event-Based Concurrency

- [ ] event-loop.md — 이벤트 기반 실행 루프
- [ ] callback.md — 완료 시 호출되는 함수
- [ ] io-multiplexing.md — select/poll/epoll 기반 다중화
- [ ] reactor-pattern.md — 이벤트 디스패치 패턴

## Chapter 12 — Asynchronous Communication

- [ ] async.md — 비동기 실행 모델 개요
- [ ] future.md — 비동기 결과를 담는 객체
- [ ] cooperative-multitasking.md — 작업이 스스로 제어권을 넘기는 멀티태스킹

## Chapter 13 — Writing Concurrent Applications

- [ ] fosters-methodology.md — 병렬 프로그램 설계 방법론

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
