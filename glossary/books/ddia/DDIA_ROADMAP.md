# DDIA Glossary 진도표
참고: Designing Data-Intensive Applications — Martin Kleppmann (O'Reilly, 2017)

---

## Part I — Foundations of Data Systems

### Chapter 1 — Reliable, Scalable, and Maintainable Applications

- [x] reliability.md — 하드웨어/소프트웨어/휴먼 에러에도 올바르게 동작하는 성질
- [x] scalability.md — 부하 증가에 대응하는 시스템의 능력, load/performance 기술 방법
- [x] maintainability.md — operability / simplicity / evolvability 세 축
- [x] latency-vs-response-time.md — latency와 response time의 차이, percentile(p99 등) 의미
- [x] throughput.md — 단위 시간당 처리량, 배치 시스템에서의 의미

### Chapter 2 — Data Models and Query Languages

- [x] relational-model.md — 테이블/관계 기반 모델, SQL의 토대
- [x] document-model.md — JSON/BSON 기반 모델, locality 장점과 join 한계
- [x] schema-on-write-vs-read.md — 스키마 강제 시점 차이, 각각의 트레이드오프
- [x] impedance-mismatch.md — 객체와 관계형 모델 간 불일치 문제
- [x] graph-model.md — 노드/엣지 기반 모델, many-to-many 관계에 강한 이유
- [x] declarative-vs-imperative-query.md — SQL(선언형) vs 명령형 쿼리의 차이

### Chapter 3 — Storage and Retrieval

- [x] hash-index.md — 인메모리 해시맵 기반 인덱스, Bitcask 방식
- [x] sstable.md — Sorted String Table, 키 정렬 보장 세그먼트 파일
- [x] lsm-tree.md — Log-Structured Merge Tree, write-optimized 저장 구조
- [x] b-tree.md — 페이지 단위 읽기/쓰기, 대부분 DB의 기본 인덱스 구조
- [x] write-amplification.md — 하나의 논리적 쓰기가 여러 번 물리 쓰기를 유발하는 현상
- [x] compaction.md — LSM-Tree에서 세그먼트를 병합해 공간 회수하는 과정
- [x] oltp-vs-olap.md — 트랜잭션 처리(OLTP)와 분석 처리(OLAP)의 차이
- [x] column-oriented-storage.md — 컬럼 단위 저장, OLAP 쿼리 최적화 원리
- [x] data-warehouse.md — 분석용 별도 저장소, ETL로 OLTP에서 데이터 적재

### Chapter 4 — Encoding and Evolution

- [ ] encoding.md — 인메모리 객체를 바이트 시퀀스로 변환하는 것 (직렬화)
- [ ] schema-evolution.md — 스키마 변경 시 하위/상위 호환성 유지 방법
- [ ] backward-forward-compatibility.md — backward(구버전이 신버전 데이터 읽기) / forward(신버전이 구버전 데이터 읽기)
- [ ] protobuf-thrift-avro.md — 바이너리 인코딩 포맷 비교, 필드 태그와 스키마 처리 방식
- [ ] dataflow.md — 데이터가 시스템 간 흐르는 방식 (DB / 서비스 / 메시지 큐)

---

## Part II — Distributed Data

### Chapter 5 — Replication

- [ ] replication.md — 같은 데이터를 여러 노드에 복사하는 것, 목적과 방식
- [ ] leader-follower-replication.md — 단일 리더가 쓰기 처리, 팔로워가 복제
- [ ] synchronous-vs-asynchronous-replication.md — 동기/비동기 복제의 내구성-가용성 트레이드오프
- [ ] replication-lag.md — 비동기 복제에서 팔로워가 리더보다 뒤처지는 현상
- [ ] read-your-own-writes.md — 자신이 쓴 데이터를 바로 읽을 수 있다는 일관성 보장
- [ ] monotonic-reads.md — 시간이 거꾸로 가는 읽기를 방지하는 일관성 보장
- [ ] multi-leader-replication.md — 여러 노드가 쓰기를 받는 구조, 충돌 해결 문제
- [ ] leaderless-replication.md — 리더 없이 quorum으로 읽기/쓰기 일관성 확보 (Dynamo 스타일)
- [ ] quorum.md — 분산 시스템에서 다수결로 일관성을 보장하는 메커니즘

### Chapter 6 — Partitioning

- [ ] partitioning.md — 데이터를 여러 노드에 분산 저장하는 것 (sharding)
- [ ] partition-by-key-range.md — 키 범위 기반 파티셔닝, 범위 쿼리에 유리
- [ ] partition-by-hash.md — 해시 기반 파티셔닝, 균등 분산에 유리
- [ ] hot-spot.md — 특정 파티션에 부하가 집중되는 현상
- [ ] secondary-index-partitioning.md — 파티션 환경에서 보조 인덱스 처리 방식 (document vs term 기반)
- [ ] rebalancing.md — 노드 추가/제거 시 파티션을 재분배하는 과정

### Chapter 7 — Transactions

- [ ] transaction.md — 여러 읽기/쓰기를 하나의 논리 단위로 묶는 것
- [ ] acid.md — Atomicity / Consistency / Isolation / Durability 각각의 의미
- [ ] isolation-level.md — read committed / snapshot isolation / serializability 비교
- [ ] read-committed.md — dirty read 방지, dirty write 방지 보장
- [ ] snapshot-isolation.md — 트랜잭션 시작 시점의 스냅샷을 읽는 방식 (MVCC)
- [ ] mvcc.md — Multi-Version Concurrency Control, 읽기와 쓰기가 서로 블록하지 않는 원리
- [ ] write-skew.md — 두 트랜잭션이 각각 유효한 쓰기를 했지만 결합하면 제약 위반이 되는 현상
- [ ] phantom-read.md — 트랜잭션 중간에 다른 트랜잭션의 삽입으로 결과 집합이 달라지는 현상
- [ ] serializability.md — 트랜잭션이 직렬로 실행된 것과 동일한 결과를 보장하는 가장 강한 격리
- [ ] two-phase-locking.md — 2PL, 읽기/쓰기 락으로 serializability 구현하는 전통적 방법

### Chapter 8 — The Trouble with Distributed Systems

- [ ] partial-failure.md — 분산 시스템에서 일부만 실패하고 나머지는 동작하는 상황
- [ ] unreliable-network.md — 패킷 손실/지연/중복/순서 변경이 일어나는 네트워크의 현실
- [ ] unreliable-clocks.md — 분산 시스템에서 시계를 신뢰할 수 없는 이유 (clock drift 등)
- [ ] process-pause.md — GC, 가상화, OS 스케줄링으로 프로세스가 임의로 멈추는 현상
- [ ] byzantine-fault.md — 악의적이거나 임의의 잘못된 응답을 보내는 노드가 있는 상황

### Chapter 9 — Consistency and Consensus

- [ ] linearizability.md — 시스템이 레지스터 하나처럼 동작하는 것처럼 보이는 가장 강한 일관성
- [ ] causality.md — 인과관계가 있는 이벤트 간 순서 보장
- [ ] eventual-consistency.md — 충분한 시간이 지나면 모든 노드가 같은 값으로 수렴하는 성질
- [ ] consensus.md — 여러 노드가 하나의 값에 합의하는 문제, FLP 불가능성 정리
- [ ] two-phase-commit.md — 2PC, 분산 트랜잭션의 원자적 커밋 프로토콜
- [ ] raft-paxos.md — 분산 합의 알고리즘, leader election과 log replication

---

## Part III — Derived Data

### Chapter 10 — Batch Processing

- [ ] batch-processing.md — 유한한 입력 데이터를 한꺼번에 처리하는 방식
- [ ] mapreduce.md — Map → shuffle → Reduce 기반 분산 배치 처리 모델
- [ ] distributed-filesystem.md — HDFS 같은 분산 파일 시스템, MapReduce의 저장 기반

### Chapter 11 — Stream Processing

- [ ] stream-processing.md — 무한한 이벤트 스트림을 지속적으로 처리하는 방식
- [ ] event-sourcing.md — 상태 변경을 이벤트 로그로 저장하고 현재 상태를 재구성하는 패턴
- [ ] change-data-capture.md — DB 변경사항을 이벤트 스트림으로 내보내는 기법 (CDC)
- [ ] stream-join.md — 스트림 처리에서 스트림-스트림 / 스트림-테이블 / 테이블-테이블 조인
- [ ] exactly-once.md — 메시지가 정확히 한 번만 처리됨을 보장하는 것, 구현의 어려움

### Chapter 12 — The Future of Data Systems

- [ ] derived-data.md — 원본 데이터에서 변환/집계로 만들어진 데이터, 재생성 가능
- [ ] unbundling-databases.md — DB 기능을 개별 도구로 분리해 조합하는 아키텍처 방향

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
