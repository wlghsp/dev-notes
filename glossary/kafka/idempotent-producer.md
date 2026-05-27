# Idempotent Producer

Kafka 프로듀서가 메시지를 재전송해도 브로커에 중복이 쌓이지 않도록 보장하는 기능.

`enable.idempotence=true`로 활성화한다. 활성화하면 `acks=all`, `retries=Integer.MAX_VALUE`가 자동으로 설정된다.

## 동작 원리

프로듀서가 시작될 때 브로커로부터 고유한 Producer ID(PID)를 발급받는다. 이후 각 메시지에 PID와 단조 증가하는 Sequence Number를 붙여서 보낸다.

```
메시지: { PID=42, SeqNum=7, data=... }
```

브로커는 파티션별로 마지막으로 받은 SeqNum을 기억한다. 다음 메시지의 SeqNum이 기대값이 아니면 두 가지로 판단한다.

- SeqNum이 이미 본 값이면 → 중복, 무시
- SeqNum이 기대값보다 크게 건너뛰면 → 유실, 오류 반환

## 보장 범위

프로듀서 → 브로커 구간에서만 Exactly-Once를 보장한다.

같은 파티션 내에서만 유효하다. 프로듀서가 재시작되면 PID가 새로 발급되기 때문에, 재시작 전후의 중복은 막지 못한다. 재시작 이전 메시지의 중복은 소비자 측 deduplication이 필요하다.

## Idempotent Producer만으로는 Exactly-Once가 안 된다

소비자 → 처리 → 결과 발행까지 원자적으로 묶으려면 Kafka 트랜잭션이 추가로 필요하다.

참고: kafka-acks.md
참고: kafka-transactions.md
참고: idempotency.md
参考: deduplication.md
