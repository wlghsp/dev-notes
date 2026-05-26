# Debezium MySQL 권한 해석

Debezium이 MySQL에 연결할 때 요구하는 6개 권한의 목적과 흐름.

```sql
GRANT SELECT, LOCK TABLES, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT
  ON *.* TO 'debezium'@'%';
```

## 권한별 역할

**SELECT**
테이블 스냅샷 초기 읽기. Debezium은 처음 연결할 때 대상 테이블 전체를 읽어 Kafka로 보낸다.

**LOCK TABLES**
스냅샷 시 일관된 읽기를 보장하기 위해 테이블을 잠근다. 잠금 없이 읽으면 읽는 도중 데이터가 바뀔 수 있다.

**RELOAD**
`FLUSH TABLES WITH READ LOCK` 실행 권한. 스냅샷 시점을 binlog 위치와 정확히 맞추기 위해 필요하다.

**SHOW DATABASES**
모니터링 대상 데이터베이스 목록을 조회한다.

**REPLICATION SLAVE**
binlog 스트림을 슬레이브처럼 읽는 핵심 권한. 이게 없으면 CDC 자체가 안 된다.

**REPLICATION CLIENT**
`SHOW MASTER STATUS`, `SHOW SLAVE STATUS`로 현재 binlog 위치를 확인한다.

## 단계별 흐름

초기 스냅샷: SELECT + LOCK TABLES + RELOAD 조합으로 일관된 시점의 전체 데이터를 읽는다.

이후 CDC: REPLICATION SLAVE로 binlog를 스트리밍한다. 여기서부터는 SELECT 없이도 동작한다.

위치 확인: REPLICATION CLIENT로 현재 binlog 파일명과 offset을 조회한다. 재시작 후 어디서부터 이어 읽을지 파악하는 데 쓴다.

## 핵심

REPLICATION SLAVE가 없으면 binlog를 못 읽고, SELECT가 없으면 초기 스냅샷이 안 된다. 6개 중 하나라도 빠지면 Debezium 연결 단계 또는 스냅샷 단계에서 실패한다.
