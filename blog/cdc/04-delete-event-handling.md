# DELETE 이벤트 처리: tombstone과 rewrite 모드

## DELETE가 특별한 이유

INSERT와 UPDATE는 단순하다. 변경 후 데이터(`after`)가 있으니까 메시지로 만들면 된다.

DELETE는 다르다. row가 사라졌다는 사실을 어떻게 메시지로 표현할 것인가. 그리고 그 메시지를 받은 Sink는 어떻게 DELETE를 수행할 것인가. 이 두 가지가 까다롭다.

---

## Debezium의 DELETE 메시지 흐름

Debezium은 DELETE 발생 시 두 개의 메시지를 순서대로 발행한다.

**1. DELETE 이벤트 메시지**

```json
{
  "before": { "PRJCT_ID": "abc123", "BIZ_NM": "테스트" },
  "after": null,
  "op": "d"
}
```

**2. Tombstone 메시지**

key는 동일하고 value가 null인 메시지. Kafka의 log compaction을 위한 것이다. 이 메시지가 있어야 compaction 시 해당 key의 모든 과거 메시지가 정리된다.

---

## tombstones.on.delete 설정

```json
"tombstones.on.delete": "true"
```

기본값이 true다. false로 설정하면 tombstone을 발행하지 않는다. log compaction을 쓰지 않는다면 false도 무방하지만, 운영 환경에서는 compaction을 위해 true를 유지하는 게 맞다.

---

## ExtractNewRecordState의 delete.handling.mode

`ExtractNewRecordState` transform 적용 시 DELETE를 어떻게 처리할지 결정하는 옵션이다.

- `drop` — DELETE 이벤트를 버린다
- `rewrite` — `__deleted: true` 필드를 추가해서 유지한다
- `none` — 기본 구조 그대로 유지

```json
"transforms.unwrap.delete.handling.mode": "rewrite"
```

`rewrite` 모드를 쓰면 DELETE 이벤트가 이렇게 바뀐다:

```json
{
  "PRJCT_ID": "abc123",
  "__op": "d",
  "__deleted": "true"
}
```

Sink에서 `__deleted`가 `"true"`인 메시지를 받으면 DELETE를 실행하면 된다.

---

## Logstash에서 DELETE 처리

실제 Logstash filter에서 이렇게 분기했다:

```ruby
if [__deleted] == "true" {
  # DELETE 처리
  conn.createStatement.execute("DELETE FROM #{table} WHERE #{pk_col} = '#{pk_val}'")
} else {
  # UPSERT 처리
  # INSERT INTO ... ON DUPLICATE KEY UPDATE ...
}
```

`__op`가 아닌 `__deleted`를 기준으로 분기한 이유는 `rewrite` 모드에서 `__deleted`가 명시적으로 `"true"` / `"false"` 문자열로 오기 때문에 조건이 명확하다.

---

## drop.tombstones 설정

```json
"transforms.unwrap.drop.tombstones": "false"
```

tombstone 메시지(value=null)를 파이프라인에서 통과시킬지 결정한다. `false`로 설정하면 tombstone이 그대로 흘러간다.

Sink에서 null 메시지를 처리 못한다면 `true`로 설정해서 버려도 된다. 단, Kafka log compaction이 제대로 동작하려면 tombstone이 Kafka 토픽에는 있어야 한다 — Sink 파이프라인에서 버리는 건 상관없다.

---

## 다음

[05 - 양방향 동기화와 무한루프 방지](05-bidirectional-sync-loop-prevention.md)
