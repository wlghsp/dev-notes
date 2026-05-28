# Shard (샤드)

인덱스를 물리적으로 나눈 단위. Elasticsearch가 데이터를 여러 노드에 분산 저장하기 위한 기본 단위다.

하나의 Shard는 독립적인 Lucene 인덱스다. 검색과 색인이 Shard 단위로 실행된다.

## Primary Shard vs Replica Shard

Primary Shard: 실제 데이터를 처음 저장하는 원본 샤드. 색인 요청은 항상 Primary로 먼저 간다.

Replica Shard: Primary의 복제본. 장애 시 Primary를 대체하고, 읽기 요청을 분산 처리해 검색 처리량을 높인다.

참고: replica.md

## 왜 나누는가

단일 노드 하나에 모든 데이터를 담으면 두 가지 문제가 생긴다.

1. 용량 한계: 데이터가 노드 하나의 디스크를 초과할 수 있다.
2. 성능 한계: 검색 요청이 하나의 노드에 집중된다.

Shard로 나누면 여러 노드에 데이터를 분산하고 검색을 병렬 처리할 수 있다.

## 라우팅

문서가 색인될 때 어떤 Shard에 저장될지는 아래 공식으로 결정된다.

```
shard_num = hash(document_id) % number_of_primary_shards
```

기본적으로 문서 ID의 해시값을 Shard 수로 나눈 나머지로 결정된다. Primary Shard 수가 고정인 이유가 여기에 있다 — 중간에 바뀌면 기존 문서의 라우팅 결과가 달라져 찾을 수 없게 된다.

## Shard 크기

권장 크기는 10GB ~ 50GB 사이. 너무 크면 복구 시간이 길어지고, 너무 작으면 Shard 관리 오버헤드가 커진다.

참고: index.md, replica.md, node.md, cluster.md
