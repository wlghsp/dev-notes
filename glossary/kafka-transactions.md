# Kafka Transactions

소비 → 처리 → 발행을 하나의 원자적 단위로 묶어서, 중간에 실패하면 전체를 롤백하는 기능.

Kafka 내부에서 Exactly-Once를 구현하는 핵심 메커니즘이다.

## 왜 필요한가

Idempotent Producer는 프로듀서 → 브로커 구간의 중복을 막는다. 그런데 소비자가 메시지를 읽고, 처리하고, 결과를 다른 토픽에 발행하는 과정 전체를 원자적으로 보장하려면 더 필요하다.

처리는 완료됐는데 결과 발행 전에 죽으면, 재시작 후 같은 메시지를 다시 처리하고 결과를 중복 발행한다. 트랜잭션은 이 시나리오를 막는다.

## 동작 흐름

```
transactional.id 설정 → Transaction Coordinator로부터 PID 발급
BEGIN TRANSACTION
  consume(input-topic, offset=5)
  process()
  produce(output-topic, result)
  commitOffset(input-topic, offset=6)
END TRANSACTION (commit)
```

트랜잭션이 커밋되기 전까지 output-topic에 발행된 메시지는 다른 소비자에게 보이지 않는다. 커밋이 완료돼야 보인다.

실패하면 abort되고, 발행된 메시지는 무효화된다. 소비자는 abort된 메시지를 받지 않는다.

## 소비자 측 설정

소비자가 커밋된 메시지만 읽으려면 `isolation.level=read_committed`로 설정해야 한다.

기본값은 `read_uncommitted`다. 이 상태에서는 아직 커밋되지 않은 트랜잭션 메시지도 읽힌다. 트랜잭션을 쓰는 의미가 없어진다.

## transactional.id

트랜잭션 프로듀서를 식별하는 고유한 문자열이다.

프로듀서가 재시작해도 같은 transactional.id를 쓰면 이전 트랜잭션 상태를 이어받는다. 덕분에 재시작 이전의 미완료 트랜잭션을 abort하고 새로 시작할 수 있다.

## 보장 범위

Kafka 내부, 즉 토픽 간 메시지 이동에서만 Exactly-Once를 보장한다.

Kafka → 외부 DB처럼 시스템 경계를 넘는 순간 트랜잭션 보장이 깨진다. 외부 시스템에 쓰는 작업은 별도로 idempotency나 deduplication을 적용해야 한다.

참고: idempotent-producer.md
참고: kafka-acks.md
참고: exactly-once.md
참고: consumer-group.md
