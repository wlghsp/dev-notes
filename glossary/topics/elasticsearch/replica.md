# Replica (레플리카)

Primary Shard의 복제본. 가용성과 읽기 성능을 높이기 위해 존재한다.

## 역할

가용성: Primary Shard가 있는 노드가 다운되면 Replica가 새로운 Primary로 승격된다. 데이터 유실 없이 서비스를 유지할 수 있다.

읽기 성능: 검색 요청은 Primary와 Replica 모두에 분산된다. Replica가 많을수록 읽기 처리량이 늘어난다.

## 쓰기 흐름

색인 요청은 Primary Shard에 먼저 처리되고, 이후 모든 Replica에 복제된다. Primary가 성공하고 Replica 복제가 완료된 후에야 클라이언트에 응답을 돌려준다.

## Primary와 같은 노드에 배치 불가

Elasticsearch는 Primary Shard와 그 Replica를 같은 노드에 두지 않는다. 노드 장애 시 Primary와 Replica를 동시에 잃는 상황을 방지하기 위해서다.

노드가 1개뿐이면 Replica를 배치할 곳이 없어 `yellow` 상태가 된다 — 데이터는 있지만 복제가 안 된 상태.

## Replica 수 조정

Replica 수는 운영 중에도 변경 가능하다.

```json
PUT /products/_settings
{
  "number_of_replicas": 2
}
```

Primary Shard 수와 달리 Replica 수는 언제든 바꿀 수 있다.

참고: shard.md, node.md, cluster.md
