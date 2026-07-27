# debezium-offset-recovery

**"Debezium이 재시작할 때 어디서부터 다시 읽는지, 그 지점 계산이 어떤 경우에 어긋나는지"**

debezium.md에서 offset은 binlog 파일명과 position이라고 정리했다. 여기서는 그 offset이 실제로 어디에 저장되고, 재시작 시점에 어떻게 복구되며, 어떤 상황에서 유실이나 중복이 발생하는지를 다룬다.

---

## offset이 저장되는 곳

Debezium은 자체 상태 저장소가 없다. Kafka Connect가 관리하는 내부 토픽(`offset.storage.topic`)에 커넥터별 offset을 커밋한다.

```
Debezium Task가 binlog 읽음
  → 일정 이벤트마다 offset(파일명 + position)을 Kafka Connect에 보고
  → Kafka Connect가 offset.storage.topic에 커밋
```

이 토픽은 컴팩션(compaction)이 걸려 있어서, 같은 커넥터의 최신 offset만 남고 이전 값은 정리된다. 즉 "마지막으로 커밋된 offset" 하나만 신뢰할 수 있는 값이다.

---

## 재시작 시 복구 순서

Task가 재시작하면 순서는 이렇다.

1. Kafka Connect가 offset.storage.topic에서 해당 커넥터의 마지막 offset을 읽는다.
2. Debezium이 그 offset(파일명 + position)으로 MySQL에 복제 연결을 요청한다.
3. MySQL이 해당 position부터 binlog 이벤트를 스트리밍한다.

이 복구가 성립하려면 전제가 하나 있다. MySQL 서버에 그 binlog 파일이 아직 남아 있어야 한다는 것이다.

---

## offset 유실 시나리오

### 1. binlog 파일이 이미 삭제된 경우

MySQL은 `expire_logs_days` (또는 `binlog_expire_logs_seconds`) 설정에 따라 오래된 binlog 파일을 자동 삭제한다. Debezium이 오래 멈춰 있다가 재시작했는데, 마지막 offset이 가리키는 binlog 파일이 이미 삭제됐다면 그 position부터 이어 읽을 수 없다.

이 경우 Debezium은 `ERROR: Cannot replicate because the master purged required binary logs` 형태로 실패한다. 복구하려면 스냅샷을 다시 찍어야 하고, 그 사이 발생한 변경 이벤트 중 스냅샷으로 커버되지 않는 구간(순수 delete 이력 등)은 유실된다.

이유: Debezium은 MySQL 입장에서 Replica다. 실제 Replica가 오래 끊겨서 필요한 binlog가 지워지는 것과 동일한 문제를 겪는다.

### 2. offset 커밋 주기와 재처리 중복

Debezium은 이벤트를 매번 처리할 때마다 offset을 커밋하지 않는다. `offset.flush.interval.ms` 주기로 배치 커밋한다. 커밋 직전에 Task가 죽으면, 마지막 커밋 이후 이미 Kafka로 발행된 이벤트들이 다음 재시작 시 다시 발행된다.

```
이벤트 A 발행 → 이벤트 B 발행 → (커밋 예정 시점) 죽음
  → 재시작 시 마지막 커밋된 offset부터 다시 읽음
  → A, B가 다시 발행됨 (중복)
```

이것이 debezium.md에서 언급한 At-Least-Once의 실제 발생 지점이다. 커밋 주기를 짧게 하면 중복 범위는 줄지만, MySQL과 Kafka Connect에 커밋 요청이 그만큼 잦아진다.

### 3. Task 재분배와 offset 불일치

Connect 클러스터에서 Task가 다른 Worker로 재분배(rebalance)되면, 새 Worker는 offset.storage.topic에서 offset을 다시 읽어와 이어받는다. 이 시점에 offset.storage.topic 자체가 아직 최신 상태로 복제되지 않았다면(컨슈머 랙), 오래된 offset을 읽어 불필요한 재처리가 발생할 수 있다.

---

## 정리

offset 유실은 "Debezium이 offset을 잘못 계산해서"가 아니라, offset이 가리키는 binlog가 MySQL에서 더 이상 존재하지 않을 때 발생한다. 운영 관점에서 확인할 지점은 두 가지다.

- `expire_logs_days`가 Debezium의 최대 다운타임보다 충분히 긴가
- offset.flush.interval.ms가 다운스트림의 멱등 처리 능력과 맞는 수준인가

---

## 한 줄 요약

> offset 복구는 Kafka Connect 내부 토픽의 마지막 offset으로 MySQL에 재연결을 시도하는 것이며, 그 시점에 필요한 binlog가 이미 삭제됐으면 스냅샷 재수행 없이는 복구할 수 없다.

참고: debezium.md
참고: debezium-snapshot.md
참고: cdc.md
참고: kafka-connect.md
참고: kafka-connect-rebalancing.md
