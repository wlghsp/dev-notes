# Index (인덱스)

Elasticsearch에서 데이터를 저장하는 논리적 단위. 관계형 DB의 테이블에 해당한다.

하나의 인덱스는 같은 성격의 Document들을 담는다. 예를 들어 `products`, `orders`, `users`처럼 도메인별로 인덱스를 분리한다.

## 구성

인덱스는 물리적으로 여러 Shard로 나뉘어 저장된다. 인덱스 자체는 논리적 개념이고, 실제 데이터는 Shard 안의 Segment에 있다.

```
Index: products
  ├── Shard 0 (Primary)  → Node A
  ├── Shard 1 (Primary)  → Node B
  ├── Shard 0 (Replica)  → Node B
  └── Shard 1 (Replica)  → Node A
```

## 인덱스 생성

```json
PUT /products
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "price": { "type": "integer" }
    }
  }
}
```

`number_of_shards`: 인덱스를 몇 개의 Primary Shard로 나눌지. 생성 후 변경 불가.
`number_of_replicas`: 각 Primary Shard의 복제본 수. 운영 중 변경 가능.

## 주의점

Shard 수는 인덱스 생성 시 결정되고 이후 변경할 수 없다. 처음부터 데이터 규모를 고려해 설정해야 한다. 너무 많으면 오버헤드, 너무 적으면 스케일아웃이 어렵다.

참고: shard.md, replica.md, mapping.md, segment.md
