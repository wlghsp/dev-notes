# CS:APP Glossary 진도표
참고: Computer Systems: A Programmer's Perspective (3rd Edition) — Bryant & O'Hallaron

---

## Chapter 1 — A Tour of Computer Systems

- [x] compilation-pipeline.md — preprocessor → compiler → assembler → linker 흐름
- [x] cache.md — L1/L2/L3 캐시, 지역성(locality)
- [x] storage-hierarchy.md — 레지스터부터 디스크까지의 계층 구조
- [x] process.md — OS가 관리하는 실행 단위, 추상화 개념
- [x] thread.md — 프로세스 내 독립적인 실행 흐름
- [x] virtual-memory.md — 프로세스가 보는 가상 주소 공간
- [x] amdahls-law.md — 병렬화 성능 상한, 직렬 구간이 천장을 만든다
- [x] fork-exec.md — 프로세스 복제/교체, OS 프로세스 추상화의 핵심 예시

## Chapter 2 — Representing and Manipulating Information

- [x] byte-ordering.md — big endian / little endian
- [x] unsigned-encoding.md — unsigned 정수 표현, B2U 공식
- [x] twos-complement.md — 2의 보수, sign bit, 표현 범위
- [x] integer-overflow.md — wrap-around, undefined behavior
- [x] bitwise-operations.md — AND/OR/XOR/NOT, 비트 마스킹
- [x] bit-shift.md — 논리/산술 시프트, 2의 거듭제곱 곱셈
- [x] ieee-floating-point.md — IEEE 754 구조, bias, NaN, Infinity
- [x] floating-point-rounding.md — round-to-even, 비결합성

## Chapter 3 — Machine-Level Representation of Programs

- [x] machine-code.md — 기계어 vs 어셈블리, ISA 추상화, ATT/Intel 형식
- [x] x86-64-registers.md — 16개 범용 레지스터, 역할 분담, 하위 비트 접근
- [x] operand-types.md — immediate/register/memory 오퍼랜드, 주소 계산 공식
- [x] mov-instruction.md — 데이터 이동, zero/sign extension, movl 암묵적 동작
- [x] push-pop.md — 스택 push/pop 동작 원리, %rsp 변화
- [x] leaq-instruction.md — 주소 계산 명령어, 컴파일러의 산술 최적화 활용
- [x] condition-codes.md — CF/ZF/SF/OF 플래그, CMP/TEST/SET 명령어
- [x] control-flow.md — jump, if-else/while/switch의 어셈블리 구현
- [x] cmov-instruction.md — 조건부 이동, branch prediction miss 회피
- [x] array-access.md — 배열 메모리 레이아웃, 인덱스 계산, 다차원 배열
- [x] struct-memory.md — 필드 오프셋, alignment, 패딩, union
- [x] stack-frame.md — 함수 호출 시 스택에 생성되는 프레임 구조
- [x] calling-convention.md — 함수 호출 규약, 레지스터 역할 분담
- [x] stack-canary.md — 스택 보호 메커니즘, canary 삽입과 검사
- [x] buffer-overflow.md — 스택 버퍼 오버플로우와 보안 취약점

## Chapter 4 — Processor Architecture

- [ ] instruction-set-architecture.md — ISA 개념, 프로그래머가 보는 CPU 인터페이스
- [ ] pipeline.md — CPU 파이프라이닝, 스테이지 분리

## Chapter 5 — Optimizing Program Performance

- [ ] locality.md — 시간적/공간적 지역성
- [ ] branch-prediction.md — 분기 예측, 미스 패널티
- [ ] loop-unrolling.md — 루프 언롤링 최적화 기법

## Chapter 6 — The Memory Hierarchy

- [ ] sram-dram.md — SRAM vs DRAM 구조 차이
- [ ] cache-miss.md — cold/capacity/conflict miss
- [ ] direct-mapped-cache.md — 직접 매핑 캐시
- [ ] set-associative-cache.md — 집합 연관 캐시

## Chapter 7 — Linking

- [ ] static-linking.md — 컴파일 타임 링킹
- [ ] dynamic-linking.md — 런타임 공유 라이브러리 링킹
- [ ] symbol-table.md — 링커가 참조하는 심볼 테이블
- [ ] position-independent-code.md — PIC, 주소 독립 코드

## Chapter 8 — Exceptional Control Flow

- [ ] exception.md — trap / fault / abort 분류
- [ ] signal.md — Unix 시그널 메커니즘
- [x] context-switch.md — 프로세스/스레드 컨텍스트 스위치
- [ ] fork-exec.md — 프로세스 생성, 8장 Exceptional Control Flow 맥락
- [ ] system-call.md — 시스템 콜 메커니즘, user/kernel 모드 전환

## Chapter 9 — Virtual Memory

- [ ] page-table.md — 가상→물리 주소 매핑 자료구조
- [ ] page-fault.md — 페이지 폴트 처리 흐름
- [ ] tlb.md — Translation Lookaside Buffer
- [ ] memory-mapping.md — 파일/익명 메모리 매핑
- [ ] heap-allocator.md — malloc/free 동작 원리
- [ ] fragmentation.md — 내부/외부 단편화
- [ ] page-cache.md — 파일 시스템과 메모리 사이의 캐시 계층

## Chapter 10 — System-Level I/O

- [ ] io-redirection.md — stdin/stdout 리다이렉션
- [ ] buffered-io.md — 커널 버퍼 vs 유저스페이스 버퍼
- [ ] file-descriptor.md — 파일 디스크립터, 열린 파일 테이블 구조

## Chapter 11 — Network Programming

- [ ] ip-address.md — IP 주소 체계, 도메인 이름
- [ ] http.md — HTTP 트랜잭션, 웹 서버 기초
- [ ] socket.md — 소켓 인터페이스, 네트워크 연결 추상화
- [ ] tcp.md — TCP 연결 관리, 신뢰성 있는 데이터 전송

## Chapter 12 — Concurrent Programming

- [ ] mutex.md — 뮤텍스, 임계구역 보호
- [ ] semaphore.md — 세마포어, 카운팅 동기화
- [ ] race-condition.md — 경쟁 조건, 데이터 레이스
- [ ] deadlock.md — 교착 상태 발생 조건
- [ ] io-multiplexing.md — select/poll/epoll 기반 다중화
- [ ] concurrency.md — 동시성 개념, 스레드/프로세스/I/O 기반 동시성

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
