# replication-lag-impact

**"MySQL 복제 지연이 CDC 파이프라인의 지연으로 어떻게 이어지는가"**

cdc.md에서 Debezium은 MySQL 입장에서 Replica처럼 동작한다고 정리했다. 그렇다면 실제 Replica가 겪는 복제 지연 문제를 Debezium도 똑같이 겪는지, 겪는다면 어느 지점에서인지를 짚는다.

---

## 오해하기 쉬운 지점

"복제 지연(replication lag)"이라고 하면 보통 Primary-Replica 사이의 지연을 떠올린다. Debezium은 별도 DB 인스턴스가 아니라 binlog를 구독하는 클라이언트이므로, 전통적인 Replica의 복제 지연과는 지연이 발생하는 위치가 다르다.

CDC 파이프라인에서 지연은 세 구간으로 나뉜다.

```
1구간: 트랜잭션 커밋 → binlog에 기록되는 시점까지
2구간: binlog 기록 → Debezium이 읽어서 Kafka에 발행하는 시점까지
3구간: Kafka 발행 → 다운스트림 컨슈머가 소비하는 시점까지
```

"복제 지연이 CDC 지연으로 이어진다"는 표현은 주로 1구간과 2구간을 가리킨다.

---

## 1구간 — 커밋과 binlog 기록 사이

MySQL은 트랜잭션 커밋 시 binlog에 동기적으로 기록한다(`sync_binlog` 설정에 따라 디스크 flush 시점은 달라짐). 이 자체는 지연이 거의 없다.

다만 Primary에 별도의 실제 Replica가 존재하고 그 Replica의 복제가 지연되고 있는 상황이라면, 이는 Debezium과 무관한 문제다. Debezium은 Primary의 binlog를 직접 읽으므로 실제 Replica의 지연에 영향받지 않는다. 이 지점이 "복제 지연"이라는 용어 때문에 가장 흔히 헷갈리는 부분이다.

---

## 2구간 — Debezium이 실제로 지연되는 지점

Debezium이 겪는 지연은 아래 요인에서 발생한다.

- **대량 트랜잭션 처리**: 한 번에 수만 건을 변경하는 배치 트랜잭션이 있으면, binlog 이벤트를 순차 처리하는 Debezium이 그 구간 동안 뒤처진다.
- **스냅샷 중**: 초기 스냅샷(debezium-snapshot.md)이 진행되는 동안은 실시간 binlog 반영이 뒤로 밀린다.
- **Kafka 발행 지연**: Kafka 브로커 자체가 느리거나(디스크 I/O, 네트워크), 파티션 리더 변경 중이면 Debezium의 발행이 막힌다.
- **rebalancing 공백**: Task가 재배정되는 동안 binlog를 읽지 않는다(kafka-connect-rebalancing.md).

즉 "복제 지연이 CDC 지연을 만든다"는 표현보다는, "Debezium 자신의 처리 속도가 binlog 생성 속도를 못 따라가면 지연이 쌓인다"가 더 정확하다.

---

## 측정 방법

Debezium은 `MilliSecondsBehindSource`라는 메트릭을 제공한다. 마지막으로 처리한 이벤트의 소스 타임스탬프와 현재 시각의 차이다. 이 값이 지속적으로 증가하면 Debezium이 binlog 생성 속도를 따라가지 못하고 있다는 뜻이다.

이 값이 튀는 시점을 위 세 구간(대량 트랜잭션 / 스냅샷 / Kafka 발행 / rebalancing) 중 어디와 겹치는지 대조하면 원인을 좁힐 수 있다.

---

## 한 줄 요약

> CDC 지연은 Primary-Replica 간 전통적 복제 지연이 아니라, Debezium 자신이 binlog를 읽고 Kafka에 발행하는 속도가 변경 발생 속도를 못 따라갈 때 생긴다.

참고: cdc.md
참고: debezium-snapshot.md
참고: kafka-connect-rebalancing.md
