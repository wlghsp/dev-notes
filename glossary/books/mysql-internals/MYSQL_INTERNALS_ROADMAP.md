# MySQL Internals Glossary 진도표
참고: Understanding MySQL Internals — Sasha Pachev (O'Reilly, 2007)

전략: MySQL이 "왜 이렇게 구현했는가"를 이해하는 것이 목표.
DB Internals(Petrov)나 DDIA의 이론이 MySQL에서 어떻게 구체화됐는지 연결하는 관점으로 진행한다.

---

## Chapter 1 — MySQL History and Architecture

MySQL 전체 구조를 한 번에 잡는 챕터. 이후 모든 챕터의 지도가 된다.

- [x] mysql-architecture.md — MySQL의 주요 레이어 구조. 클라이언트 → 쿼리 파서 → 옵티마이저 → 스토리지 엔진
- [x] mysql-source-modules.md — 소스코드 주요 모듈과 역할. sql/, storage/, mysys/ 등 디렉토리 구조
- [x] ch01-mysql-history-and-architecture.md — 챕터 1 종합 문서

---

## Chapter 3 — Core Classes, Structures, and APIs

MySQL 내부 코드를 이해하는 데 필요한 핵심 자료구조.

- [x] thd.md — Thread Descriptor. 스레드 하나의 상태 전체를 담는 핵심 구조체
- [x] net.md — Network Connection Descriptor. 클라이언트/서버 프로토콜 구현의 핵심
- [x] mysql-table-structure.md — TABLE 클래스. 열린 테이블의 메타데이터와 런타임 상태
- [x] field-class.md — Field 클래스. 컬럼 하나를 표현하는 추상 기반 클래스. 타입별 서브클래스
- [x] ch03-core-classes-structures-apis.md — 챕터 3 종합 문서

---

## Chapter 4 — Client/Server Communication

클라이언트가 쿼리를 보내고 결과를 받기까지의 통신 흐름.

- [ ] mysql-protocol.md — MySQL 클라이언트/서버 프로토콜 개요. 패킷 포맷
- [ ] mysql-handshake.md — 클라이언트 연결 시 인증 핸드셰이크 과정
- [ ] mysql-command-packet.md — 클라이언트가 서버에 보내는 커맨드 패킷 구조. COM_QUERY 등

---

## Chapter 6 — Thread-Based Request Handling

MySQL이 다중 클라이언트를 어떻게 동시에 처리하는가.

- [ ] mysql-thread-model.md — 왜 프로세스가 아닌 스레드를 사용하는가. 스레드 기반 모델의 장단점
- [ ] mysql-connection-thread.md — 커넥션 하나당 스레드 하나. 연결 수립부터 쿼리 처리까지의 흐름
- [ ] mysql-thread-cache.md — 스레드 재사용을 위한 캐시. thread_cache_size의 의미

---

## Chapter 7 — The Storage Engine Interface

MySQL의 가장 독특한 설계: 스토리지 엔진 플러그인 구조.

- [ ] storage-engine-interface.md — handler 추상 클래스. MySQL 코어와 스토리지 엔진 사이의 계약
- [ ] handler-api.md — ha_write_row / ha_read_row 등 핵심 handler API. 코어가 엔진을 호출하는 방식
- [ ] storage-engine-pluggable.md — 왜 플러그인 구조인가. InnoDB / MyISAM / Memory 엔진이 같은 인터페이스를 구현하는 이유

---

## Chapter 8 — Concurrent Access and Locking

MySQL에서 동시 접근을 제어하는 락 메커니즘.

- [ ] mysql-lock-types.md — 테이블 락 / 행 락 / 메타데이터 락의 차이. 각 스토리지 엔진이 어떤 락을 사용하는가
- [ ] table-lock-manager.md — MySQL 코어의 테이블 락 매니저. MDL(Metadata Lock) 개념
- [ ] mysql-lock-vs-innodb-lock.md — MySQL 서버 레벨 락과 InnoDB 레벨 락의 관계. 두 계층이 존재하는 이유

---

## Chapter 9 — Parser and Optimizer

SQL 쿼리가 실행 계획으로 바뀌는 과정.

- [ ] mysql-parser.md — SQL 텍스트를 파싱해 내부 AST로 변환하는 과정. Lex/Yacc 기반
- [ ] mysql-optimizer.md — 옵티마이저가 실행 계획을 선택하는 방식. cost-based 최적화의 기초
- [ ] query-execution-flow.md — 클라이언트 쿼리가 파싱 → 최적화 → 실행 → 결과 반환까지 거치는 전체 흐름
- [ ] execution-plan.md — EXPLAIN 출력과 내부 실행 계획의 연결. rows / type / key 각 항목의 의미

---

## Chapter 10 — Storage Engines

주요 스토리지 엔진의 구조와 특성.

- [ ] innodb-overview.md — InnoDB 아키텍처 개요. 버퍼 풀, 클러스터드 인덱스, MVCC, 트랜잭션 지원
- [ ] myisam-vs-innodb.md — MyISAM과 InnoDB의 근본적 차이. 트랜잭션 / 락 / 크래시 복구
- [ ] innodb-buffer-pool.md — InnoDB의 핵심. 디스크 페이지를 메모리에 캐시하는 구조
- [ ] innodb-clustered-index.md — InnoDB에서 데이터가 PK 순서로 저장되는 구조. 세컨더리 인덱스와의 관계

---

## Chapter 11 — Transactions

트랜잭션을 지원하기 위해 스토리지 엔진이 구현해야 하는 것들.

- [ ] mysql-transaction-architecture.md — MySQL 코어와 스토리지 엔진 사이에서 트랜잭션이 어떻게 조율되는가
- [ ] innodb-mvcc.md — InnoDB의 MVCC 구현. undo log를 이용한 스냅샷 읽기
- [ ] innodb-undo-log.md — 트랜잭션 롤백과 MVCC를 위한 undo log 구조
- [ ] innodb-redo-log.md — 크래시 복구를 위한 redo log. WAL(Write-Ahead Logging) 원리
- [ ] deadlock-detection.md — InnoDB의 데드락 감지 알고리즘과 자동 롤백

---

## Chapter 12 — Replication

MySQL 복제가 내부적으로 어떻게 동작하는가.

- [ ] mysql-replication-overview.md — 복제의 기본 구조. 소스 → 바이너리 로그 → 레플리카
- [ ] binlog.md — 바이너리 로그 포맷. statement-based vs row-based 복제의 차이
- [ ] replication-thread.md — IO 스레드와 SQL 스레드. 레플리카 측 복제 처리 흐름

---

진행 방식: Chapter 1 → 3 → 6 → 7 → 8 → 9 → 10 → 11 → 12 순서 권장.
완료된 항목은 [x]로 표시.
