# Cluster (클러스터)

여러 Node가 함께 작동하는 Elasticsearch의 논리적 단위. 같은 `cluster.name`을 가진 노드들이 자동으로 하나의 클러스터를 형성한다.

## 역할

단일 노드로는 처리할 수 없는 데이터 규모와 트래픽을 여러 노드에 분산해 처리한다. 노드 장애 시에도 Replica Shard 덕분에 서비스를 유지한다.

## 클러스터 상태(Cluster Health)

Elasticsearch는 클러스터 상태를 세 가지 색으로 표현한다.

Green: 모든 Primary Shard와 Replica Shard가 정상 할당된 상태. 완전히 정상.

Yellow: 모든 Primary Shard는 할당됐지만 일부 Replica Shard가 할당되지 못한 상태. 데이터는 있고 검색도 되지만, 노드 장애 시 데이터 유실 위험이 있다. 노드가 1개뿐일 때 Replica를 설정하면 이 상태가 된다.

Red: 일부 Primary Shard가 할당되지 못한 상태. 해당 Shard의 데이터를 색인하거나 검색할 수 없다.

## 클러스터 상태 확인

```
GET /_cluster/health
```

```json
{
  "status": "green",
  "number_of_nodes": 3,
  "number_of_data_nodes": 3,
  "active_primary_shards": 15,
  "active_shards": 30,
  "unassigned_shards": 0
}
```

## 클러스터 메타데이터

Master Node가 클러스터 전체의 메타데이터를 관리한다. 인덱스 목록, 각 Shard가 어느 노드에 있는지, 노드 목록 등이 포함된다. 이 정보를 Cluster State라고 부른다.

참고: node.md, shard.md, replica.md, shard-allocation.md
