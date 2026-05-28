# WSREP (Write-Set Replication)

Galera Cluster의 복제 프로토콜. 트랜잭션을 Write-Set 단위로 클러스터 전체에 전파한다.

## Write-Set이란

트랜잭션이 변경한 행들의 집합 + 트랜잭션 메타데이터를 묶은 단위다.

트랜잭션이 커밋을 시도하면 MySQL은 변경 내용을 Write-Set으로 직렬화해서 다른 모든 노드에 브로드캐스트한다. 각 노드는 이 Write-Set을 받아 충돌 여부를 검증하고, 문제 없으면 자신의 스토리지에 적용한다.

## 흐름

```
Node1에서 트랜잭션 커밋 시도
  → Write-Set 생성
  → 모든 노드에 브로드캐스트 (wsrep_provider)
  → 각 노드가 Certification 검증
  → 충돌 없음 → 전체 노드 적용, 커밋 성공
  → 충돌 있음 → 충돌난 트랜잭션 롤백
```

## 동기 복제의 의미

WSREP는 "virtually synchronous"다. 트랜잭션 커밋 시점에 모든 노드가 Write-Set을 받았음을 보장하지만, 모든 노드가 실제로 디스크에 적용했음을 보장하지는 않는다.

받았다(received) ≠ 적용됐다(applied). 이 차이가 장애 복구 시나리오에서 중요하다.

## wsrep_local_recv_queue

각 노드는 받은 Write-Set을 큐에 쌓아두고 순서대로 적용한다. 이 큐가 쌓이면 노드가 클러스터 속도를 따라가지 못하는 것이다. `wsrep_local_recv_queue` 값이 지속적으로 높으면 해당 노드에 부하가 있다는 신호다.

참고: galera-cluster.md
참고: certification-based-replication.md
