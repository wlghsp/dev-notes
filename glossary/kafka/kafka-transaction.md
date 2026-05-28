# kafka-transaction

**"여러 토픽/파티션에 대한 produce를 하나의 원자적 단위로 묶는 것"**

DB 트랜잭션이 여러 row 변경을 원자적으로 묶듯이, Kafka 트랜잭션은 여러 토픽/파티션에 대한 메시지 발행을 원자적으로 묶는다. 커밋 전까지 소비자에게 보이지 않는다.

---

## DB 트랜잭션과의 차이

DB 트랜잭션은 데이터 변경의 일관성을 보장한다. Kafka 트랜잭션은 메시지 발행의 원자성을 보장한다. 목적은 비슷하지만 대상이 다르다.

소비자가 트랜잭션 메시지를 보려면 `isolation.level=read_committed`로 설정해야 한다. 기본값은 `read_uncommitted`라서 커밋 전 메시지도 읽힌다.

---

## 주된 쓰임새 — Exactly-Once

```
Kafka Consumer → 메시지 처리 → Kafka Producer로 결과 발행
```

이 흐름에서 "소비 + 발행"을 하나의 트랜잭션으로 묶으면, 중간에 장애가 나도 둘 다 커밋되거나 둘 다 롤백된다. 중복 발행 없이 Exactly-Once를 구현할 수 있다.

---

## 시스템 경계에서는 보장이 깨진다

Kafka 트랜잭션은 Kafka 내부에서만 유효하다. Kafka → 외부 DB에 쓰는 작업은 트랜잭션 범위 밖이다.

```
Kafka 트랜잭션 범위
  └── 토픽 A에서 소비
  └── 토픽 B에 발행
      ↓ 여기서 경계가 끊김
  외부 DB에 INSERT  ← Kafka 트랜잭션과 묶을 수 없음
```

이 경계를 넘는 순간 Exactly-Once 보장이 어렵다. 소비자 측의 멱등 처리나 중복 제거가 별도로 필요하다.

---

## 한 줄 요약

> Kafka 트랜잭션 = Kafka 내부에서 소비 + 발행을 원자적으로 묶어 Exactly-Once를 구현하는 수단. Kafka 밖으로 나가면 보장이 끊긴다.

참고: exactly-once.md
참고: idempotency.md
참고: cdc.md
