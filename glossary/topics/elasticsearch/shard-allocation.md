# Shard Allocation (샤드 할당)

Master Node가 각 Shard를 어느 Data Node에 배치할지 결정하는 과정.

## 기본 원칙

Elasticsearch는 Shard를 자동으로 분산 배치한다. 기본 규칙은 두 가지다.

1. 같은 인덱스의 Primary Shard와 그 Replica는 같은 노드에 두지 않는다.
2. 노드 간 Shard 수를 최대한 균등하게 분배한다.

## 할당이 일어나는 시점

- 인덱스 생성 시 Primary Shard 최초 배치
- 노드 장애 시 Replica가 새 Primary로 승격하고 새 Replica 배치
- 새 노드 추가 시 기존 Shard를 재분배(Rebalancing)
- 노드 제거 시 해당 노드의 Shard를 다른 노드로 이동

## Unassigned Shard

배치되지 못한 Shard가 생기면 Cluster Health가 Yellow 또는 Red가 된다.

원인은 주로 세 가지다.
- 노드 수보다 Shard 수가 많아 배치할 곳이 없는 경우
- 노드 장애로 Primary Shard가 사라진 경우
- 디스크 공간 부족으로 할당을 거부한 경우

## 할당 제어

할당 정책은 다양한 설정으로 제어할 수 있다.

특정 노드에 Shard를 배치하지 않도록 제외하거나, 노드 속성(rack, zone)을 기준으로 Shard를 분산할 수 있다. 운영 중 노드를 순차적으로 재시작할 때 할당을 일시 중단하는 것도 가능하다.

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

참고: node.md, cluster.md, shard.md, rebalancing.md, replica.md
