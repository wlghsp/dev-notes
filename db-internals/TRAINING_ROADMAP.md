# DB Internals 트레이닝 로드맵
## "내부 구조를 이해하는 개발자 되기"

> 특정 DB 문서를 외우는 게 목표가 아니다. **왜 이렇게 동작하는지 설명할 수 있는 수준**이 목표다.
> 각 Phase는 문서 읽기 → 실습 → 이해도 테스트 → 블로그 발행 순서를 지킨다.

---

## 진행 원칙

```
실습 없이 다음 Phase로 넘어가지 않는다
블로그를 발행하지 않으면 완료가 아니다
"대충 알 것 같다"는 완료가 아니다 — 설명할 수 있어야 완료다
특정 DB 지식 이전에 "왜"를 먼저 이해한다
```

---

## Phase 구성 전략

```
Phase 0~4 : DB 공통 내부 원리 (특정 제품에 종속되지 않는 개념)
             → Database Internals (Alex Petrov) Part I 기반
Phase 5~6 : MySQL/MariaDB 기반 심화 실습
Phase 7   : 실전 트러블슈팅
```

---

## Phase 현황

| Phase | 주제 | 실습 | 블로그 발행 | 발표 |
|---|---|---|---|---|
| 0 | 스토리지 기초 — Page, Buffer Pool, I/O | ✅ | ✅ | ✅ |
| 1 | DBMS 아키텍처 — Storage Engine은 어떻게 구성되는가 | ⬜ | ⬜ | ⬜ |
| 2 | B-Tree 내부 구조 — 왜 DB는 B-Tree를 선택했는가 | ⬜ | ⬜ | ⬜ |
| 3 | LSM-Tree — 쓰기 최적화 구조의 원리 | ⬜ | ⬜ | ⬜ |
| 4 | 트랜잭션 & MVCC | ⬜ | ⬜ | ⬜ |
| 5 | WAL & Crash Recovery | ⬜ | ⬜ | ⬜ |
| 6 | 인덱스 내부 구조 & 쿼리 실행 엔진 | ⬜ | ⬜ | ⬜ |
| 7 | 실전 트러블슈팅 | ⬜ | ⬜ | ⬜ |

---

## 각 Phase 상세

### Phase 0: 스토리지 기초 — 데이터는 디스크에 어떻게 저장되는가
**파일**: [`db-storage-basics.md`](db-storage-basics.md)
**완료 기준**: "Page가 뭔지, Buffer Pool이 왜 필요한지 동료에게 5분 설명 가능"
**핵심 질문**:
- 왜 DB는 바이트 단위가 아닌 Page 단위로 읽고 쓰는가?
- Disk I/O가 왜 병목인가? Random I/O vs Sequential I/O의 차이는?
- Buffer Pool이 없으면 무슨 일이 생기는가?
- Dirty Page를 즉시 디스크에 쓰지 않는 이유는?
**실습**:
- MySQL `innodb_page_size` 확인 + 실제 `.ibd` 파일 크기 관찰
- `SHOW ENGINE INNODB STATUS`로 Buffer Pool hit rate 확인
- `innodb_buffer_pool_size` 조정 전후 쿼리 속도 비교

---

### Phase 1: DBMS 아키텍처 — Storage Engine은 어떻게 구성되는가
**파일**: `db-dbms-architecture.md` (미작성)
**완료 기준**: "쿼리가 들어와서 결과가 나올 때까지 어떤 컴포넌트를 거치는지 그림으로 설명 가능"
**핵심 질문**:
- DBMS는 어떤 컴포넌트로 구성되는가? (Transport → Query Processor → Execution Engine → Storage Engine)
- Storage Engine 내부의 Transaction Manager / Lock Manager / Buffer Manager / Recovery Manager는 각각 무슨 역할인가?
- Row-oriented vs Column-oriented DB의 차이는? 언제 어느 쪽이 유리한가?
- In-memory DB와 Disk-based DB는 어떻게 다른가?
**실습**:
- MySQL에서 Storage Engine 목록 확인 (`SHOW ENGINES`)
- InnoDB vs MyISAM 기본 특성 비교
- `EXPLAIN`으로 쿼리가 어떤 경로로 실행되는지 관찰

---

### Phase 2: B-Tree 내부 구조 — 왜 DB는 B-Tree를 선택했는가
**파일**: `db-btree-internals.md` (미작성)
**완료 기준**: "B-Tree 노드가 Page와 어떻게 연결되는지, 삽입 시 Page split이 왜 일어나는지 설명 가능"
**핵심 질문**:
- B-Tree는 왜 Binary Search Tree가 아닌가? 디스크 구조와 어떻게 맞물리는가?
- B-Tree 노드(Page)에 데이터가 가득 찼을 때 무슨 일이 일어나는가? (Page split)
- B-Tree에서 삭제 시 Page merge는 언제 발생하는가?
- B+Tree와 B-Tree의 차이는? MySQL은 왜 B+Tree를 쓰는가?
**실습**:
- 대량 순차 삽입 vs 랜덤 삽입 성능 비교 (Page split 영향 확인)
- `innodb_space` 도구로 B-Tree 페이지 구조 시각화

---

### Phase 3: LSM-Tree — 쓰기 최적화 구조의 원리
**파일**: `db-lsm-tree.md` (미작성)
**완료 기준**: "B-Tree와 LSM-Tree의 트레이드오프를 workload 관점에서 설명 가능"
**핵심 질문**:
- LSM-Tree는 왜 쓰기가 빠른가? (Sequential Write, Memtable)
- SSTable이란 무엇인가?
- Compaction이란 무엇이고 왜 필요한가?
- Write-heavy 서비스에서 B-Tree 대신 LSM-Tree를 선택하는 기준은?
**참고 DB**: RocksDB, Cassandra, LevelDB 개념 비교 (MySQL은 RocksDB 기반 MyRocks도 지원)

---

### Phase 4: 트랜잭션 & MVCC
**파일**: `db-transaction-mvcc.md` (미작성)
**완료 기준**: "MVCC가 어떻게 Lock 없이 일관된 읽기를 제공하는지 그림으로 설명 가능"
**핵심 질문**:
- ACID에서 Isolation이 실제로 어떻게 구현되는가?
- MVCC에서 같은 row의 여러 버전은 어디에 저장되는가? (Undo Log)
- Phantom Read는 왜 발생하고, MySQL은 이를 어떻게 막는가? (Gap Lock)
- Isolation Level별로 실제 동작이 어떻게 달라지는가?
**실습**:
- MySQL에서 Isolation Level 바꿔가며 Dirty Read / Non-Repeatable Read 직접 재현
- `SHOW ENGINE INNODB STATUS`로 Lock 상황 관찰
- Deadlock 직접 발생시키고 로그 읽기

---

### Phase 5: WAL & Crash Recovery
**파일**: `db-wal-crash-recovery.md` (미작성)
**완료 기준**: "서버가 갑자기 꺼져도 데이터가 보존되는 이유를 WAL 흐름으로 설명 가능"
**핵심 질문**:
- WAL(Write-Ahead Log)이란 무엇인가? 왜 데이터보다 로그를 먼저 쓰는가?
- MySQL의 InnoDB Redo Log vs Binlog — 각각의 역할은?
- Checkpoint란 무엇이고 왜 필요한가?
- Crash 후 DB가 재시작될 때 어떤 순서로 복구되는가?
**실습**:
- MySQL Redo Log 파일(`ib_logfile0`) 위치 확인 및 크기 변경
- `innodb_flush_log_at_trx_commit` 값 변경 후 성능 vs 내구성 트레이드오프 측정
- 강제 crash 후 복구 과정 관찰 (`kill -9` + 재시작)

---

### Phase 6: 인덱스 내부 구조 & 쿼리 실행 엔진
**파일**: `db-index-and-query.md` (미작성)
**완료 기준**: "`EXPLAIN ANALYZE` 결과를 보고 실행 계획을 예측하고, 잘못된 선택을 강제로 바꿀 수 있음"
**핵심 질문**:
- Clustered Index vs Secondary Index의 차이는?
- Covering Index란 무엇이고 왜 빠른가?
- 옵티마이저는 어떤 통계 정보를 보고 실행 계획을 결정하는가?
- Join 알고리즘의 종류와 각각이 유리한 상황은? (Nested Loop, Hash Join)
- Statistics가 오래되면 왜 잘못된 실행 계획이 나오는가?
**실습**:
- `EXPLAIN ANALYZE`로 Full Scan vs Index Scan vs Covering Index 비교
- `ANALYZE TABLE`로 통계 갱신 전후 실행 계획 비교
- `USE INDEX` / `FORCE INDEX`로 실행 계획 강제 변경

---

### Phase 7: 실전 트러블슈팅
**파일**: `db-troubleshooting.md` (미작성)
**완료 기준**: "슬로우 쿼리 로그를 받아서 병목 지점을 찾고 개선 방향을 제시할 수 있음"
**핵심 질문**:
- Slow Query Log에서 어떤 정보를 봐야 하는가?
- Lock wait timeout이 발생하면 어디서부터 조사하는가?
- Connection pool 고갈은 왜 생기고 어떻게 모니터링하는가?
**실습**:
- `slow_query_log` 활성화 + `mysqldumpslow`로 분석
- `SHOW PROCESSLIST` + `INFORMATION_SCHEMA.INNODB_LOCKS`로 Lock 추적
- `pt-query-digest`로 쿼리 패턴 분석 (Percona Toolkit)

---

## 참고 자료
- *Database Internals* — Alex Petrov (O'Reilly, 2019) : Part I Storage Engines 기반
- MySQL 8.0 Reference Manual
