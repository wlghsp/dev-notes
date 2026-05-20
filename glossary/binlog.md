# Binlog (Binary Log)

MariaDB/MySQL 고유 구현으로, 데이터를 변경하는 모든 이벤트를 순서대로 기록하는 로그 파일. 어떤 쿼리가 언제, 어떤 순서로 실행됐는지가 담겨 있다. (유사 개념: PostgreSQL WAL, Oracle Redo Log, SQL Server Transaction Log)

이름에 "binary"가 붙은 이유는 사람이 읽기 어려운 바이너리 형식으로 저장되기 때문이다. `mysqlbinlog` 도구로 사람이 읽을 수 있는 형태로 변환해서 볼 수 있다.

---

## 무엇을 기록하는가

데이터를 바꾸는 이벤트만 기록한다. SELECT는 기록하지 않는다.

기록되는 것들:
- INSERT, UPDATE, DELETE
- DDL (CREATE TABLE, DROP TABLE, ALTER TABLE 등)
- 트랜잭션 커밋

binlog는 "무엇이 변경됐는가"의 기록이기 때문에 복제와 복구 모두에서 사용된다.

---

## 왜 중요한가

**복제의 기반**  
Master-Slave 복제는 binlog를 통해 이루어진다. Master가 binlog를 쓰면, Slave의 IO 스레드가 그 binlog를 읽어와서 relay log에 저장하고, SQL 스레드가 relay log를 재실행(replay)하는 방식이다. binlog 없이는 복제가 불가능하다.

**Point-in-Time Recovery**  
특정 시점의 full dump에 그 이후 binlog를 순서대로 replay하면 원하는 시점의 데이터 상태로 복구할 수 있다. 예를 들어 실수로 테이블을 DROP했을 때, 직전 dump + binlog replay로 되돌릴 수 있다.

---

## Binlog Format

binlog에 이벤트를 어떻게 기록할지 세 가지 방식이 있다.

STATEMENT — 실행된 SQL 문 자체를 기록한다. 용량이 작지만 `NOW()`, `RAND()` 같은 비결정적 함수가 포함된 쿼리는 Master와 Slave에서 다른 결과를 낼 수 있다.

ROW — 변경된 행의 실제 데이터를 기록한다. 용량이 크지만 정확하다. 현재 기본값이며 권장 방식이다.

MIXED — 대부분은 STATEMENT, 비결정적 함수가 포함된 경우는 ROW로 기록한다.

---

## GTID와 Binlog의 관계

GTID(Global Transaction ID)는 binlog에 기록된 각 트랜잭션에 붙는 고유 식별자다. `도메인ID-서버ID-시퀀스번호` 형태로 표현된다 (예: `0-1-12345`).

GTID 없이 복제하면 Slave가 "binlog 파일명 + position"으로 위치를 추적한다. Master가 바뀌면 새 Master의 binlog 파일명과 position을 다시 맞춰야 해서 failover 후 복제 재연결이 번거롭다.

GTID를 쓰면 트랜잭션 단위로 추적하므로 Master가 바뀌어도 `MASTER_USE_GTID=slave_pos`만 설정하면 Slave가 알아서 이어받는다. 참고: failover.md

---

## 다른 DB의 유사 개념

binlog는 MariaDB/MySQL 고유 구현이지만, "변경 이벤트를 순서대로 기록한다"는 개념은 모든 RDBMS에 있다.

PostgreSQL의 WAL(Write-Ahead Log)은 crash recovery 목적으로 설계됐지만, Streaming Replication이 WAL을 직접 전송하는 방식이라 WAL이 복제 메커니즘 그 자체다. "활용" 수준이 아니라 복제의 핵심이다.

Oracle은 Redo Log가 있고, 복제 전용으로는 Supplemental Logging / LogMiner 개념이 따로 있다. binlog와 더 정확히 대응되는 건 Supplemental Log다.

SQL Server는 Transaction Log가 같은 역할을 한다.

MariaDB/InnoDB는 내부적으로 crash recovery용 redo log와 복제·PITR용 binlog, 두 레이어를 따로 유지한다. binlog는 복제·PITR 중심으로 설계된 별도 레이어다.

---

## 운영에서 마주치는 상황

**Slave 복제 에러**  
binlog에 기록된 DDL을 Slave에서 replay할 때 테이블이 없거나 이미 존재하면 SQL 스레드가 멈춘다. 4.28 ecp_paas 장애가 이 케이스다. 해당 GTID를 skip하거나 에러 이후 시점부터 다시 dump해서 재연결해야 한다. 참고: mariadb-replication-recovery.md

**binlog 용량 관리**  
binlog는 계속 쌓인다. `expire_logs_days` (MariaDB 10.6+는 `binlog_expire_logs_seconds`)로 자동 삭제 주기를 설정해두지 않으면 디스크를 가득 채운다.

```sql
-- 현재 설정 확인
SHOW VARIABLES LIKE 'expire_logs_days';
SHOW VARIABLES LIKE 'binlog_expire_logs_seconds';

-- 오래된 binlog 수동 삭제 (position 확인 후 신중하게)
PURGE BINARY LOGS BEFORE '2026-04-01 00:00:00';
```

**binlog 분석**  
특정 시점에 어떤 쿼리가 실행됐는지 확인할 때:

```bash
mysqlbinlog \
  --start-datetime="2026-04-28 00:00:00" \
  --stop-datetime="2026-04-28 23:59:59" \
  /var/lib/mysql/mariadb-bin.000XXX
```
