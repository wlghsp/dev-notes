# Database Internals Glossary 진도표
참고: Database Internals — Alex Petrov (O'Reilly, 2019)

전략: "왜 이렇게 설계했는가"를 이해하는 것이 목표.
스토리지 엔진의 내부 구조 → 분산 시스템으로 이어지는 흐름을 잡는다.

---

## Chapter 1 — Introduction and Overview

데이터베이스 시스템을 분류하고, 스토리지 엔진이 무엇인지 정의하는 챕터. 이후 모든 챕터의 지도.

- [x] dbms-architecture.md — DBMS의 레이어 구조. Transport → Query Processor → Execution Engine → Storage Engine
- [x] memory-vs-disk-dbms.md — 인메모리 vs 디스크 기반 DBMS의 근본적 차이와 내구성 처리 방식
- [x] column-vs-row-oriented.md — 컬럼 지향 vs 로우 지향 저장 방식. 접근 패턴에 따른 선택 기준
- [x] data-files-and-index-files.md — 데이터 파일과 인덱스 파일의 분리. IOT, 힙 파일, 기본/보조 인덱스
- [x] buffering-immutability-ordering.md — 스토리지 엔진 설계의 3가지 핵심 축
- [x] ch01-introduction-and-overview.md — 챕터 1 종합 문서

---

## Chapter 2 — B-Tree Basics

B-Tree가 왜 디스크 기반 스토리지의 표준이 됐는가.

- [ ] binary-search-tree.md — BST의 구조와 한계. 왜 디스크에서 쓸 수 없는가
- [ ] b-tree.md — B-Tree의 구조. 노드 분할/병합. 디스크 친화적 설계
- [ ] b-tree-node-format.md — 노드 내부 포맷. 키, 포인터, 데이터의 배치

---

## Chapter 3 — File Formats

디스크에 데이터를 어떻게 직렬화해서 저장하는가.

- [ ] page-layout.md — 페이지 구조. 슬롯 배열과 셀 배치
- [ ] slotted-pages.md — 슬롯 페이지. 가변 길이 레코드를 페이지 안에 배치하는 방법
- [ ] cell-format.md — 셀의 포맷. 키-값 쌍을 바이트로 직렬화하는 방식

---

## Chapter 4 — Implementing B-Trees

B-Tree를 실제로 구현할 때 마주치는 문제들.

- [ ] b-tree-lookup.md — B-Tree 탐색 알고리즘. 루트에서 리프까지
- [ ] b-tree-insert.md — 삽입과 노드 분할. 오버플로우 처리
- [ ] b-tree-delete.md — 삭제와 노드 병합. 언더플로우 처리
- [ ] b-tree-rebalance.md — 재균형. 형제 노드 빌림과 병합의 선택 기준

---

## Chapter 5 — Transaction Processing and Recovery

스토리지 엔진이 트랜잭션과 복구를 어떻게 처리하는가.

- [ ] wal.md — Write-Ahead Log. 로그 먼저 쓰는 이유와 복구 원리
- [ ] concurrency-control.md — 동시성 제어. MVCC와 락의 역할
- [ ] recovery.md — 크래시 복구. redo/undo 로그 처리 흐름

---

## Chapter 6 — B-Tree Variants

B-Tree를 확장하거나 변형한 구조들.

- [ ] copy-on-write-tree.md — 쓰기 시 복사 B-Tree. 불변성을 이용한 동시성 처리
- [ ] lazy-b-tree.md — Lazy B-Tree. 인메모리 버퍼로 I/O를 줄이는 방법
- [ ] fw-tree.md — FD-Tree / Fractional Cascading. 쓰기 최적화 변형

---

## Chapter 7 — Log-Structured Storage

불변성과 순차 쓰기를 이용한 스토리지 엔진.

- [ ] lsm-tree.md — LSM Tree 구조. 인메모리 버퍼 + 정렬된 디스크 레벨
- [ ] sstable.md — Sorted String Table. LSM의 디스크 레벨 단위
- [ ] compaction.md — 컴팩션. 여러 SSTable을 병합하는 전략

---

진행 방식: Chapter 1 → 2 → 3 → 4 → 5 → 6 → 7 순서 권장.
완료된 항목은 [x]로 표시.
