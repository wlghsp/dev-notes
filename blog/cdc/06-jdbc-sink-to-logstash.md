# JDBC Sink에서 Logstash로 전환한 이유

## 처음 구조

초기 구현은 Kafka Connect 생태계 안에서 완결되는 구조였다.

```
Debezium Source Connector → Kafka → JDBC Sink Connector → MariaDB
```

JDBC Sink Connector는 Confluent가 만든 커넥터로, Kafka 메시지를 DB에 직접 쓰는 역할이다. INSERT/UPDATE는 upsert로, DELETE는 tombstone으로 처리한다.

실제로 INSERT/UPDATE/DELETE 테스트까지 통과했다.

---

## 문제가 생긴 부분

양방향 동기화를 고려하면서 요구사항이 복잡해졌다.

**sql_log_bin=0 세션 설정**이 필요했다. JDBC Sink Connector는 커넥션을 풀로 관리하는데, 커넥션 생성 시 세션 변수를 설정하는 공식적인 방법이 없다. `connection.init.sql` 같은 옵션이 있긴 한데 드라이버나 버전에 따라 동작이 다르고, 실제로 세션 단위로 적용되는지 보장이 어렵다.

반면 Logstash의 `jdbc` output이나 커스텀 Ruby 코드는 커넥션을 직접 제어할 수 있다. `conn.createStatement.execute("SET sql_log_bin=0")`을 커넥션 생성 직후에 확실하게 실행할 수 있다.

**센터별 조건 분기**도 이유 중 하나였다. Logstash는 filter 블록에서 조건을 자유롭게 쓸 수 있다. 필드 값에 따라 다른 테이블에 쓰거나, 특정 조건이면 스킵하거나 — 이런 로직을 JDBC Sink Connector에서 구현하려면 SMT(Single Message Transform)를 조합해야 하는데 한계가 있다.

---

## Logstash가 더 적합했던 이유

JDBC Sink Connector는 "설정만으로 DB에 쓰기"에 최적화되어 있다. 표준적인 단방향 동기화라면 충분하다.

Logstash는 파이프라인 안에서 **코드 수준의 제어**가 가능하다. Ruby filter로 커넥션을 직접 다루고, 조건 분기를 마음대로 넣을 수 있다. 복잡한 요구사항이 생길수록 Logstash 쪽이 유리했다.

결국 선택 기준은 단순했다: **제어권이 얼마나 필요한가.**

---

## 트레이드오프

Logstash로 전환하면서 잃은 것도 있다.

- Kafka Connect 생태계 안에서 관리 통일이 안 된다. Source는 Connect, Sink는 Logstash로 분리된다.
- JDBC Sink Connector가 자동으로 해주던 schema 추론, upsert 로직을 직접 구현해야 한다.
- Logstash 파이프라인 코드를 관리해야 한다.

하지만 운영 환경의 실제 요구사항을 충족하는 데는 Logstash가 더 적합했다.

---

## 정리

JDBC Sink Connector는 설정만으로 DB에 쓰는 표준적인 단방향 동기화에 적합하다. 세션 변수 제어가 어렵고 조건 분기는 SMT 조합으로만 가능해서 복잡한 요구사항에서 한계가 있다.

Logstash는 파이프라인 코드로 커넥션을 직접 다루고 filter에서 자유롭게 분기할 수 있다. 대신 Source는 Connect, Sink는 Logstash로 관리가 분리되고 upsert 로직을 직접 구현해야 한다.

제어권이 필요한 순간 Logstash가 유리하다.
