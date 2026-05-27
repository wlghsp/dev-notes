# cdc

**"데이터베이스의 변경 사항을 실시간으로 감지해서 다른 시스템에 전달하는 기법"**

CDC(Change Data Capture). DB에서 INSERT/UPDATE/DELETE가 발생했을 때, 그 변경을 이벤트로 캡처해서 다른 시스템(Kafka, 검색엔진, 캐시 등)에 전파한다.

---

## 왜 필요한가

여러 시스템이 같은 데이터를 봐야 할 때 문제가 생긴다.

```
주문 DB에 INSERT
  → Kafka에도 이벤트 보내야 함
  → Elasticsearch에도 색인해야 함
  → 캐시도 무효화해야 함
```

애플리케이션이 직접 각 시스템에 쓰면 일부 실패 시 데이터 불일치가 생긴다. CDC는 DB 변경 자체를 소스로 삼아서 이 문제를 해결한다.

---

## 동작 방식 — binlog 기반

가장 널리 쓰이는 방식은 DB의 binlog(binary log)를 읽는 것이다. MySQL/MariaDB는 복제를 위해 모든 변경을 binlog에 순서대로 기록한다. CDC 도구가 슬레이브처럼 binlog를 구독해서 변경 이벤트를 추출한다.

```
애플리케이션 → DB 변경
  → DB가 binlog에 기록
  → CDC 도구(Debezium 등)가 binlog 읽음
  → 변경 이벤트 생성 → Kafka 토픽으로 발행
  → 소비자(Elasticsearch, 캐시 등)가 이벤트 수신
```

애플리케이션 코드를 수정하지 않아도 된다. DB 레벨에서 변경을 잡기 때문이다.

---

## binlog와 Redo Log의 차이

헷갈리기 쉬운 부분이다.

- Redo Log — InnoDB 내부 복구용. 크래시 후 재시작 시 사용. 외부에서 읽지 않음
- binlog — MySQL 서버 레벨의 복제/감사용 로그. 슬레이브 복제와 CDC가 이걸 읽음

CDC는 Redo Log가 아니라 binlog를 읽는다.

---

## Debezium

CDC의 대표적인 오픈소스 도구. Kafka Connect 위에서 동작하며 MySQL, PostgreSQL, MongoDB 등의 binlog/oplog를 읽어서 Kafka 토픽으로 발행한다.

```
Debezium MySQL Connector
  → MySQL binlog 구독 (슬레이브처럼 동작)
  → 변경 이벤트를 Kafka 토픽으로 발행
  → before/after 상태를 JSON으로 제공
```

---

## 전달 보장과 CDC

CDC 파이프라인은 기본적으로 At-Least-Once다.

Debezium이 binlog를 읽고 Kafka에 발행할 때, 발행 후 offset을 커밋하기 전에 죽으면 재시작 시 같은 이벤트를 다시 발행한다. 소비자가 이미 처리한 이벤트를 또 받을 수 있다.

이 중복을 처리하는 방법은 두 가지다.

1. 소비자 측을 멱등하게 설계한다. 같은 이벤트가 두 번 들어와도 결과가 달라지지 않도록.
2. Deduplication을 추가한다. 이벤트 ID(binlog position 등)를 기준으로 이미 처리한 이벤트를 무시.

CDC에서 Exactly-Once를 구현하려면 Kafka 트랜잭션과 소비자의 idempotent 처리를 함께 맞춰야 하는데, 시스템 경계(Kafka → 외부 DB)에서는 완전한 보장이 어렵다.

---

## 주의점

- binlog가 활성화돼 있어야 한다 (`log_bin=ON`)
- binlog 포맷이 ROW여야 정확한 변경 내용을 캡처할 수 있다 (`binlog_format=ROW`)
- CDC 도구가 죽었다 재시작해도 어디까지 읽었는지 offset을 관리해야 중복/유실이 없다

---

## 한 줄 요약

> CDC = DB 변경을 binlog에서 읽어 이벤트로 전파. 애플리케이션 수정 없이 여러 시스템에 데이터 동기화.

참고: redo-log.md, mvcc.md
참고: at-least-once.md
참고: exactly-once.md
참고: idempotency.md
참고: deduplication.md
참고: offset.md
