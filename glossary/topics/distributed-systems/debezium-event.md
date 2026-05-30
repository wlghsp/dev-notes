# debezium-event

**"변경 전후 상태를 before/after로 담은 이벤트 구조"**

Debezium이 Kafka로 발행하는 이벤트는 단순히 "무언가 바뀌었다"가 아니다. 무엇이 어떻게 바뀌었는지를 before와 after로 명시한다.

---

## 기본 구조

```json
{
  "before": { "id": 1, "name": "철수", "age": 29 },
  "after":  { "id": 1, "name": "철수", "age": 30 },
  "op": "u",
  "source": { "db": "mydb", "table": "users", "ts_ms": 1716900000000 }
}
```

- `op` — 변경 종류
- `before` — 변경 전 row 상태
- `after` — 변경 후 row 상태
- `source` — 어느 DB/테이블/시점인지. binlog position도 포함

---

## op 값의 의미

- `c` (create) — INSERT. before는 null, after에 삽입된 row
- `u` (update) — UPDATE. before에 이전 상태, after에 새 상태
- `d` (delete) — DELETE. before에 삭제된 row, after는 null
- `r` (read) — 스냅샷 시 전체 SELECT. before는 null, after에 현재 row

---

## before가 있으려면

MySQL 기준으로, before 필드를 채우려면 binlog 포맷이 ROW여야 한다. STATEMENT 포맷이면 변경 전 상태를 binlog에서 알 수 없어서 before가 null이 된다.

```
binlog_format=ROW  → before/after 모두 채워짐
binlog_format=STATEMENT  → before는 null
```

소비자가 before를 활용할 계획이라면 ROW 포맷 확인이 필수다.

---

## 한 줄 요약

> Debezium 이벤트 = op + before + after. 변경 전후를 모두 담아서 소비자가 diff를 직접 알 수 있다.

참고: debezium.md
참고: debezium-snapshot.md
