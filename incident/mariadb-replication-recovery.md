# MariaDB Replication 복구 Runbook

**대상 환경:** MaxScale + MariaDB 삼중화 (Master 1 + Slave 2)  
**작성 배경:** 2026-04-28 ecp_paas DB 테이블 삭제 실패로 Slave 다운, 2026-05-12 dump 후 복구 시도했으나 1시간 30분 뒤 재다운

---

## 0. 현재 상태 확인 (가장 먼저)

**Master에서:**
```sql
SHOW MASTER STATUS\G
SHOW PROCESSLIST;
```

**Slave에서:**
```sql
SHOW SLAVE STATUS\G
```

봐야 할 필드:
- `Slave_IO_Running` — IO 스레드 상태 (Yes/No)
- `Slave_SQL_Running` — SQL 스레드 상태 (Yes/No)
- `Last_Error` / `Last_IO_Error` / `Last_SQL_Error` — 실제 에러 메시지
- `Seconds_Behind_Master` — 복제 지연 (줄어드는 추이인지 확인)

MariaDB 10.5+ 환경이면 아래 명령도 동일하게 동작한다:
```sql
SHOW REPLICA STATUS\G
```

---

## 1. 에러 원인 판별

### Case A: GTID 불일치

MariaDB의 GTID는 MySQL과 다르다. `gtid_current_pos`가 Slave가 현재 복제 중인 위치고, `gtid_binlog_pos`가 binlog에 기록된 위치다.

```sql
-- Master
SELECT @@global.gtid_binlog_pos;
SELECT @@global.gtid_current_pos;

-- Slave
SELECT @@global.gtid_current_pos;
SELECT @@global.gtid_slave_pos;
```

Slave의 `gtid_slave_pos`가 Master의 `gtid_binlog_pos`보다 앞서거나 충돌하면 GTID mismatch. → 3번으로 이동

### Case B: 삭제된 테이블에 대한 DDL 재실행

`Last_SQL_Error`에서 아래 패턴이 보이면 binlog에 삭제 실패 DDL이 남아 replay되는 것이다.

```
Error 'Table 'ecp_paas.xxx' doesn't exist' on query
```

→ 해당 트랜잭션 skip 또는 dump 재수행 선택

### Case C: binlog position 불일치 (GTID 미사용 환경)

```sql
SHOW SLAVE STATUS\G
-- Master_Log_File, Read_Master_Log_Pos 확인
-- Slave가 바라보는 파일/위치가 현재 Master와 맞는지 점검
```

---

## 2. 단기 임시 조치 — 에러 트랜잭션 Skip

데이터 불일치를 야기할 수 있으니 **임시방편**으로만 사용. 반드시 이후 데이터 검증 필요.

**MariaDB GTID 모드일 때:**

`Last_SQL_Error`에서 에러 GTID를 확인한 뒤 해당 위치로 `gtid_slave_pos`를 밀어넘긴다.

```sql
STOP SLAVE;
SET GLOBAL gtid_slave_pos='<에러 직후 GTID>';  -- 예: 0-1-12345
START SLAVE;
SHOW SLAVE STATUS\G
```

**non-GTID 환경:**
```sql
STOP SLAVE;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
START SLAVE;
SHOW SLAVE STATUS\G
```

---

## 3. 정식 조치 — dump 재수행

1시간 30분 만에 재다운된 원인은 dump 시 GTID 정보 없이 import했을 가능성이 높다.  
MariaDB는 `--set-gtid-purged` 옵션이 없고 대신 dump 파일에 `gtid_binlog_pos` 주석이 포함된다.

### dump 시 필수 옵션

```bash
mariadb-dump \
  --single-transaction \
  --master-data=2 \
  --all-databases \
  -u root -p > full_dump.sql
```

`mariadb-dump`가 없으면 `mysqldump`로도 동일하게 동작한다 (MariaDB는 mysqldump 호환).  
`--single-transaction` 없으면 dump 중 테이블 락 또는 데이터 불일치 발생.  
`--master-data=2`는 dump 파일 상단에 binlog position을 주석으로 기록해준다.

### Slave import 절차

```bash
# 1. Slave에서 복제 중지
mysql -u root -p -e "STOP SLAVE; RESET SLAVE ALL;"

# 2. dump import
mysql -u root -p < full_dump.sql

# 3. dump 파일에서 GTID position 확인
grep -i "gtid_binlog_pos\|CHANGE MASTER" full_dump.sql | head -5
```

### 복제 재연결 (GTID 환경 — 권장)

dump 파일에서 확인한 `gtid_binlog_pos` 값을 `gtid_slave_pos`에 설정한다.

```sql
SET GLOBAL gtid_slave_pos='<dump에서 확인한 값>';  -- 예: 0-1-12345

CHANGE MASTER TO
  MASTER_HOST='<master_ip>',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='<pw>',
  MASTER_USE_GTID=slave_pos;

START SLAVE;
SHOW SLAVE STATUS\G
```

`MASTER_USE_GTID=slave_pos`가 MariaDB GTID 복제의 핵심 옵션이다.  
MySQL의 `MASTER_AUTO_POSITION=1`과 다르니 혼동 주의.

### 복제 재연결 (non-GTID 환경)

dump 파일 상단에서 position 확인:
```
-- CHANGE MASTER TO MASTER_LOG_FILE='mariadb-bin.000XXX', MASTER_LOG_POS=XXXXXX;
```

```sql
CHANGE MASTER TO
  MASTER_HOST='<master_ip>',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='<pw>',
  MASTER_LOG_FILE='mariadb-bin.000XXX',
  MASTER_LOG_POS=XXXXXX;

START SLAVE;
SHOW SLAVE STATUS\G
```

---

## 4. 복구 후 검증

```sql
-- Slave IO/SQL 스레드 모두 Yes인지 확인
SHOW SLAVE STATUS\G

-- Seconds_Behind_Master가 0을 향해 줄어드는지 확인
-- 0이면 Master와 완전 동기화 완료
```

데이터 정합성 spot check:
```sql
-- Master와 Slave에서 동일하게 실행, row count 비교
SELECT COUNT(*) FROM ecp_paas.<핵심 테이블>;
```

MaxScale에서 Slave를 다시 인식하는지 확인:
```bash
maxctrl list servers
maxctrl list services
```

Slave 상태가 `Running`이고 `Slave`로 역할이 표시되면 MaxScale 레벨 복구 완료.

---

## 5. 원인 추적 — binlog 분석

4.28 당시 어떤 DDL이 기록됐는지 확인:

```bash
mysqlbinlog \
  --start-datetime="2026-04-28 00:00:00" \
  --stop-datetime="2026-04-28 23:59:59" \
  /var/lib/mysql/mariadb-bin.000XXX | grep -A10 "ecp_paas"
```

binlog 파일 위치는 `SHOW MASTER STATUS\G`의 `File` 필드에서 확인한다.

삭제 실패가 partial DDL로 binlog에 남아있으면 Slave에서 replay 시 항상 에러가 발생한다.  
해당 GTID 범위를 skip하거나 dump 기준점을 그 이후 시점으로 잡아야 한다.

---

## 6. 체크리스트 요약

dump 재수행 전:
- `--single-transaction` 옵션 포함 여부
- `--master-data=2` 옵션 포함 여부
- dump 완료 후 파일 내 `gtid_binlog_pos` 또는 CHANGE MASTER 주석 기록

import 후:
- `SHOW SLAVE STATUS\G` 즉시 확인
- `MASTER_USE_GTID=slave_pos` 옵션으로 복제 연결했는지 확인
- 1분, 5분, 30분, 1시간 30분 시점에 상태 재확인 (이전 장애가 1.5시간 뒤 재발했으므로)
- `Seconds_Behind_Master` 추이 모니터링
- `maxctrl list servers`로 MaxScale에서 Slave 상태 확인
