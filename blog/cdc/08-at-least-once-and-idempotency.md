# CDC 파이프라인과 at-least-once

## Kafka는 at-least-once가 기본이다

Kafka 자체는 메시지를 저장하고 전달하는 역할만 한다. Producer가 메시지를 보내고, Consumer가 가져간다. Kafka가 "정확히 한 번" 처리를 보장하려면 별도 설정이 필요하다.

기본 동작은 **at-least-once** — 메시지가 한 번 이상 전달될 수 있다.

---

## 왜 중복이 생기나

Debezium의 흐름을 보면 이해가 쉽다.

```
1. DB binlog 읽기
2. Kafka에 메시지 전송
3. offset 저장 (여기까지 읽었다고 기록)
```

2번과 3번 사이에 장애가 나면 문제다. 메시지는 이미 Kafka에 들어갔는데, offset은 저장이 안 된 상태다. 재시작하면 같은 binlog 위치부터 다시 읽어서 중복 전송이 발생한다.

이걸 막으려면 "Kafka에 쓰는 것"과 "offset 저장"을 원자적으로 묶어야 하는데, 두 작업이 서로 다른 시스템에 걸쳐 있어서 기본적으로 불가능하다.

---

## 해결책: Sink에서 멱등성 보장

중복 메시지를 막는 것보다, **중복이 와도 결과가 같도록** 만드는 게 현실적이다. 이를 멱등성(idempotency)이라고 한다.

**JDBC Sink Connector**

PK를 기준으로 upsert를 한다. 같은 메시지가 두 번 오면 같은 row를 두 번 upsert하는 것이므로 결과가 달라지지 않는다.

```json
"insert.mode": "upsert",
"pk.mode": "record_value",
"pk.fields": "id"
```

**Logstash → Elasticsearch**

`document_id`를 PK로 지정하면 ES가 upsert로 동작한다. 같은 ID면 덮어쓴다.

```ruby
output {
  elasticsearch {
    document_id => "%{id}"
  }
}
```

`document_id`를 설정하지 않으면 매번 새 document가 생성된다. 중복 메시지가 쌓이는 것이므로 CDC 파이프라인에서는 반드시 설정해야 한다.

**Logstash → MariaDB (jdbc output)**

INSERT 대신 INSERT ... ON DUPLICATE KEY UPDATE 또는 REPLACE INTO를 쓴다. PK 충돌 시 덮어쓰므로 중복에 안전하다.

---

## 정리

at-least-once는 피하는 게 아니라, Sink에서 흡수하는 구조다.

- Debezium은 offset/전송이 비원자적이라 중복이 생긴다 → Sink에서 멱등성으로 흡수
- JDBC Sink → upsert 모드
- Logstash → ES → document_id 설정
- Logstash → DB → upsert 쿼리

---

## 다음

[09 - (미정)](09-tbd.md)
