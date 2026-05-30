# Galera Auto Increment 설정

Galera Cluster에서 여러 노드가 동시에 AUTO_INCREMENT 값을 생성할 때 PK 충돌을 막는 설정.

## 문제

일반 MySQL은 노드가 하나라서 AUTO_INCREMENT가 순차적으로 증가한다. Galera는 노드가 여럿이고 각 노드가 독립적으로 INSERT를 처리하기 때문에, 아무 설정 없이 두면 여러 노드가 같은 AUTO_INCREMENT 값을 생성해서 PK 충돌이 일어난다.

## 해결 방법

각 노드가 서로 겹치지 않는 값을 생성하도록 간격과 시작점을 나눈다.

- `auto_increment_increment` — 값이 증가하는 간격. 보통 노드 수로 설정한다.
- `auto_increment_offset` — 각 노드의 시작점. 노드마다 다르게 설정한다.

노드가 3개인 경우:

```
auto_increment_increment = 3

Node1: auto_increment_offset = 1  →  1, 4, 7, 10, 13 ...
Node2: auto_increment_offset = 2  →  2, 5, 8, 11, 14 ...
Node3: auto_increment_offset = 3  →  3, 6, 9, 12, 15 ...
```

각 노드가 생성하는 값이 절대 겹치지 않는다.

## CDC에서 주의할 점

이 설정 때문에 CDC로 받는 이벤트의 PK가 순서대로 오지 않는다.

Node1이 id=1을 INSERT하고, Node2가 id=2를 INSERT하면 Kafka에는 거의 동시에 두 이벤트가 들어온다. 소비자가 id 순서로 정렬하거나 id가 시간 순서를 반영한다고 가정하면 안 된다.

또한 노드를 추가하거나 제거하면 `auto_increment_increment`가 바뀌어야 한다. 이미 생성된 값과 새로운 간격이 충돌하지 않도록 신중하게 계획해야 한다.

## Galera의 자동 설정

`wsrep_auto_increment_control=ON`이면 Galera가 클러스터 노드 수에 따라 이 값을 자동으로 조정한다. 기본값은 ON이다.

참고: galera-cluster.md
참고: certification-based-replication.md
참고: cdc.md
