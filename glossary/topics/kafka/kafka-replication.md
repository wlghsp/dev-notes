# Kafka Replication

파티션을 여러 브로커에 복제해서 장애가 나도 데이터를 잃지 않게 하는 메커니즘이다.

## 구조

파티션 하나는 복제 수(replication factor)만큼 여러 브로커에 복사본을 가진다. replication factor가 3이면 파티션 하나가 3개의 브로커에 저장된다.

복사본 중 하나는 Leader, 나머지는 Follower다.

```
partition-0 (replication factor = 3)
  broker-1: Leader
  broker-2: Follower
  broker-3: Follower
```

producer는 항상 Leader에게만 쓴다. consumer도 기본적으로 Leader에서 읽는다. Follower는 Leader를 복제하는 역할만 한다. 왜 복사본 중 하나만 쓰기를 받는 구조인지는 kafka-leader-follower.md 참고.

## ISR (In-Sync Replicas)

Leader와 동기화 상태를 유지하고 있는 복제본 목록이다. Follower는 Leader의 메시지를 따라잡고 있으면 ISR에 속하고, 너무 뒤처지면 제외된다. 진입/이탈 조건과 min.insync.replicas, High Watermark와의 관계는 kafka-isr.md 참고.

Leader가 죽으면 Kafka는 ISR 중 하나를 새 Leader로 선출한다. ISR에 속한 복제본은 Leader의 데이터를 거의 다 따라잡은 상태이므로, 새 Leader가 돼도 데이터 유실이 최소화된다.

ISR 밖의 Follower를 Leader로 선출하면 뒤처진 만큼 데이터가 유실된다. `unclean.leader.election.enable=true`로 설정하면 ISR 밖에서도 선출을 허용하는데, 가용성을 높이는 대신 데이터 유실 위험이 생긴다.

## acks 설정과의 관계

producer가 메시지를 보낼 때 "얼마나 많은 복제본이 받아야 성공으로 볼 건가"를 acks로 설정한다.

- `acks=0` — 브로커 응답을 기다리지 않는다. 가장 빠르지만 유실 가능성이 있다.
- `acks=1` — Leader가 받으면 성공으로 본다. Leader가 죽으면 Follower로 복제되기 전 메시지가 유실될 수 있다.
- `acks=all` — ISR 전체가 받아야 성공으로 본다. 가장 안전하지만 가장 느리다.

## 장애 시나리오

broker-1(Leader)이 죽으면 Kafka의 컨트롤러가 ISR 중 하나를 새 Leader로 선출한다. producer와 consumer는 새 Leader로 자동으로 연결된다. broker-1이 복구되면 Follower로 다시 참여해서 새 Leader를 따라잡는다.

이 과정에서 consumer는 잠깐 읽기가 중단될 수 있지만 데이터는 유실되지 않는다. ISR 기반으로 선출했기 때문이다.

참고: kafka-partition.md
참고: kafka-topic.md
참고: kafka-acks.md
참고: kafka-isr.md
참고: kafka-leader-follower.md
