# kafka-connect-rebalancing

**"Connect 클러스터에서 Connector/Task가 Worker 사이에 재분배되는 것, 그리고 그 순간 이벤트 처리가 멈추는 구간"**

kafka-connect.md에서 Worker가 Connector와 Task를 실행한다고 정리했다. 여러 Worker가 클러스터를 이루면, Worker 추가/제거/장애 시 Task를 어떻게 나눠 가질지 다시 정해야 한다. 이 재배치 과정이 rebalancing이다.

---

## 언제 발생하는가

- Worker가 클러스터에 새로 join 할 때 (스케일 아웃)
- Worker가 죽거나 응답이 끊길 때 (session timeout)
- Connector 설정이 바뀌어서 Task 개수가 재계산될 때
- 새 Connector가 등록되거나 기존 Connector가 삭제될 때

이 중 실전 운영에서 가장 자주 문제가 되는 건 두 번째, Worker 장애로 인한 비자발적(involuntary) rebalancing이다.

---

## 동작 방식 — Eager vs Incremental Cooperative

초기 Kafka Connect(2.3 이전)는 rebalancing이 시작되면 **모든** Connector와 Task를 일단 중지시키고, 새로 배정을 계산한 뒤 다시 시작했다. 이를 eager rebalancing이라 부른다.

```
Eager Rebalancing
  Worker 하나 join/leave
    → 클러스터의 모든 Task 정지 (stop-the-world)
    → 새 배정 계산
    → 전체 Task 재시작
```

문제는 관련 없는 Connector의 Task까지 전부 멈춘다는 점이다. Connector가 10개인 클러스터에서 하나가 재배정 대상이어도 나머지 9개까지 순간적으로 중단된다.

2.3부터는 Incremental Cooperative Rebalancing을 지원한다. 실제로 재배정이 필요한 Task만 멈추고 나머지는 계속 실행 상태를 유지한다.

```
Incremental Cooperative Rebalancing
  Worker 하나 join/leave
    → 영향받는 Task만 식별
    → 그 Task만 정지 → 재배정 → 재시작
    → 나머지 Task는 중단 없이 계속 실행
```

`connect.protocol=compatible` (또는 이후 기본값) 설정으로 활성화된다. 운영 중인 클러스터라면 이 프로토콜이 켜져 있는지가 rebalancing 영향 범위를 좌우한다.

---

## rebalancing 중 이벤트 처리 공백

재배정 대상이 된 Task는 정지-재시작 사이 구간 동안 binlog를 읽지 않는다. Eager 모드에서는 이 공백이 클러스터 전체 Connector로 번지고, Incremental 모드에서는 영향받는 Task로만 국한된다.

이 공백 자체는 이벤트 유실이 아니다. Debezium은 재시작 시 마지막 커밋된 offset부터 이어 읽으므로(debezium-offset-recovery.md), 공백 동안 MySQL에 쌓인 binlog는 재시작 후 따라잡는다. 문제는 두 가지다.

1. 공백이 길어지는 동안 다운스트림으로의 지연이 그대로 누적된다 (replication-lag-impact.md와는 별개로, Connect 레이어에서 발생하는 지연).
2. 공백 직전 커밋되지 않은 offset 구간은 재시작 후 다시 발행되어 중복이 생긴다.

---

## 운영 관점에서 확인할 것

- Kafka Connect 버전이 2.3 이상이고 Incremental Cooperative Rebalancing이 켜져 있는가
- Worker의 `session.timeout.ms`가 너무 짧아서 정상 Worker가 일시적 GC pause 등으로 죽은 것으로 오판되지 않는가 (불필요한 rebalancing 유발)
- Connector 개수가 많은 클러스터라면, 하나의 장애가 전체로 번지지 않는 구조인지

---

## 한 줄 요약

> Rebalancing은 Worker 변화에 따라 Task를 재배정하는 과정이며, Eager 방식은 클러스터 전체를 멈추지만 Incremental Cooperative 방식은 영향받는 Task만 멈춰서 공백 범위를 줄인다.

참고: kafka-connect.md
참고: debezium-offset-recovery.md
참고: replication-lag-impact.md
