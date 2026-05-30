# Modern Operating Systems Glossary 진도표
참고: Modern Operating Systems, 4th Edition — Andrew S. Tanenbaum, Herbert Bos (Pearson, 2023)

---

## Chapter 1 — Introduction

- [ ] process.md — 실행 중인 프로그램의 추상화, 주소 공간 + 실행 흐름
- [ ] address-space.md — 프로세스가 사용할 수 있는 메모리 주소의 집합
- [ ] system-call.md — 유저 프로그램이 커널 서비스를 요청하는 메커니즘
- [ ] kernel-mode-user-mode.md — CPU 실행 모드 구분, 모드 전환이 필요한 이유
- [ ] interrupt.md — 외부 이벤트가 CPU를 중단시키고 핸들러를 실행시키는 메커니즘
- [ ] trap.md — 소프트웨어에서 발생하는 동기적 인터럽트, system call의 구현 방식
- [ ] os-structure.md — 모놀리식 / 마이크로커널 / 하이브리드 커널 구조 비교

---

## Chapter 2 — Processes and Threads

- [ ] process-state.md — running / ready / blocked 세 상태와 전환 조건
- [ ] process-creation.md — fork/exec 모델, 부모-자식 프로세스 관계
- [ ] thread.md — 프로세스 내 독립적 실행 흐름, 스택과 레지스터만 별도
- [ ] user-thread-vs-kernel-thread.md — 유저 레벨 vs 커널 레벨 스레드, N:M 모델
- [ ] context-switch.md — CPU를 다른 프로세스/스레드에게 넘기는 과정, 비용
- [ ] interprocess-communication.md — 프로세스 간 통신 방법 (pipe, shared memory, message passing)
- [ ] race-condition.md — 공유 자원에 대한 비결정적 접근 순서로 결과가 달라지는 문제
- [ ] mutual-exclusion.md — 한 번에 하나의 프로세스만 임계 영역에 진입하도록 보장
- [ ] semaphore.md — 카운터 기반 동기화 도구, P/V 연산
- [ ] mutex.md — 이진 세마포어, 소유권 개념이 있는 락
- [ ] monitor.md — 상호 배제와 조건 변수를 묶은 고수준 동기화 구조
- [ ] deadlock.md — 프로세스들이 서로의 자원을 기다리며 영원히 블록되는 상태
- [ ] deadlock-conditions.md — 교착상태 발생의 4가지 필요충분조건 (Coffman 조건)
- [ ] deadlock-avoidance.md — Banker's Algorithm, 안전 상태 판별

---

## Chapter 3 — Memory Management

- [ ] memory-abstraction.md — 물리 주소를 직접 쓰지 않는 이유, 주소 공간 추상화
- [ ] virtual-memory.md — 실제 물리 메모리보다 큰 주소 공간을 제공하는 기법
- [ ] paging.md — 고정 크기 페이지로 메모리를 관리하는 방식
- [ ] page-table.md — 가상 주소를 물리 주소로 변환하는 자료구조
- [ ] tlb.md — Translation Lookaside Buffer, 페이지 테이블 조회 캐시
- [ ] page-fault.md — 접근한 페이지가 물리 메모리에 없을 때 발생하는 트랩
- [ ] page-replacement.md — 물리 메모리가 부족할 때 어떤 페이지를 교체할지 결정 (FIFO / LRU / Clock)
- [ ] segmentation.md — 가변 크기 세그먼트로 메모리를 관리하는 방식
- [ ] thrashing.md — 페이지 폴트가 너무 자주 발생해 실제 작업보다 페이징에 더 많은 시간을 쓰는 상태

---

## Chapter 4 — File Systems

- [ ] file-system.md — 파일과 디렉토리를 저장장치 위에 구성하는 방식
- [ ] inode.md — 파일의 메타데이터와 데이터 블록 포인터를 저장하는 자료구조
- [ ] directory.md — 파일 이름과 inode를 매핑하는 특수 파일
- [ ] hard-link-soft-link.md — 하드링크(inode 공유)와 심볼릭 링크(경로 참조)의 차이
- [ ] vfs.md — Virtual File System, 다양한 파일 시스템을 단일 인터페이스로 추상화
- [ ] buffer-cache.md — 디스크 블록을 메모리에 캐시해 I/O를 줄이는 기법
- [ ] journaling.md — 크래시 후 파일 시스템 일관성을 빠르게 복구하는 로그 기반 기법
- [ ] disk-scheduling.md — 디스크 헤드 이동을 최소화하는 I/O 요청 순서 결정 (SCAN 등)

---

## Chapter 5 — Input/Output

- [ ] io-software-layers.md — I/O 소프트웨어의 계층 구조 (인터럽트 핸들러 → 디바이스 드라이버 → 장치 독립 계층 → 유저 레벨)
- [ ] dma.md — Direct Memory Access, CPU 개입 없이 장치가 메모리에 직접 접근하는 방식
- [ ] device-driver.md — 특정 하드웨어를 OS가 다룰 수 있도록 추상화하는 소프트웨어
- [ ] interrupt-driven-io.md — I/O 완료 시 인터럽트로 CPU에 알리는 방식, polling과의 비교

---

## Chapter 6 — Deadlocks

- [ ] resource-allocation-graph.md — 자원 할당 상태를 그래프로 표현, 사이클로 교착상태 감지
- [ ] deadlock-detection.md — 교착상태 발생 후 탐지하고 복구하는 방식
- [ ] starvation.md — 특정 프로세스가 자원을 영원히 할당받지 못하는 상태

---

## Chapter 7 — Virtualization and the Cloud

- [ ] hypervisor.md — 여러 VM이 하나의 물리 머신에서 독립적으로 실행될 수 있게 하는 소프트웨어 계층
- [ ] type1-type2-hypervisor.md — 베어메탈(Type 1) vs 호스트 OS 위(Type 2) 하이퍼바이저 비교
- [ ] container.md — 커널을 공유하면서 프로세스를 격리하는 경량 가상화 (namespace + cgroup)
- [ ] paravirtualization.md — 게스트 OS가 가상화를 인지하고 하이퍼바이저에 직접 협력하는 방식

---

## Chapter 8 — Multiple Processor Systems

- [ ] smp.md — Symmetric Multiprocessing, 여러 CPU가 단일 공유 메모리에 접근하는 구조
- [ ] cache-coherence.md — 여러 CPU 캐시가 같은 메모리 위치에 대해 일관된 값을 보는 것
- [ ] memory-consistency-model.md — CPU가 메모리 읽기/쓰기 순서를 어디까지 보장하는가
- [ ] numa.md — Non-Uniform Memory Access, CPU마다 로컬 메모리 접근 속도가 다른 구조
- [ ] spin-lock.md — 락 획득 전까지 CPU를 바쁘게 돌리는 락, 짧은 임계 영역에 적합

---

## Chapter 9 — Security

- [ ] authentication.md — 사용자/프로세스의 신원을 확인하는 메커니즘
- [ ] access-control.md — 자원에 대한 접근 권한을 제어하는 정책과 메커니즘
- [ ] buffer-overflow.md — 스택/힙 버퍼 경계를 초과한 쓰기로 실행 흐름을 탈취하는 공격
- [ ] privilege-escalation.md — 낮은 권한 프로세스가 더 높은 권한을 얻는 것

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
