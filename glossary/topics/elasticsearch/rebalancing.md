# Rebalancing (리밸런싱)

클러스터에 노드가 추가되거나 제거될 때, Shard를 다시 균등하게 분배하는 작업.

## 언제 발생하나

노드 추가: 새 노드가 들어오면 기존 노드들의 Shard 일부를 새 노드로 옮긴다. 부하를 분산하기 위해서다.

노드 제거/장애: 노드가 빠지면 그 노드의 Shard를 남은 노드들에 재배치한다. Replica가 있으면 즉시 승격시키고, 새 Replica를 다른 노드에 만든다.

## 동작 방식

Master Node가 Cluster State를 보고 불균형 상태를 감지하면 자동으로 Shard를 이동시킨다. Shard 이동은 노드 간 데이터를 네트워크로 복사하는 작업이라 시간이 걸리고 I/O와 네트워크 부하가 생긴다.

## 리밸런싱 조절

대규모 클러스터에서 노드를 순차 재시작할 때, 매번 리밸런싱이 일어나면 비효율적이다. 리밸런싱을 일시 중단하고 모든 작업 후 한 번에 재활성화하는 것이 낫다.

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.rebalance.enable": "none"
  }
}
```

재시작 완료 후 `"all"`로 되돌린다.

## 리밸런싱 속도 제한

리밸런싱이 너무 빠르면 운영 트래픽에 영향을 줄 수 있다. 동시에 이동할 수 있는 Shard 수를 제한해 속도를 조절할 수 있다.

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.cluster_concurrent_rebalance": 2
  }
}
```

참고: shard-allocation.md, node.md, cluster.md, shard.md
