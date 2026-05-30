# Certification-Based Replication

Galera Cluster가 Multi-Master 충돌을 감지하고 처리하는 메커니즘.

낙관적 동시성 제어를 기반으로 한다. 쓰기를 막지 않고 일단 허용한 뒤, 커밋 시점에 충돌 여부를 검증한다.

## 동작 원리

트랜잭션이 커밋을 시도하면 Galera는 해당 트랜잭션이 변경한 행의 Primary Key 집합을 추출한다. 이것이 Write-Set의 핵심 정보다.

클러스터의 모든 노드는 이 Write-Set을 받아서 "같은 시간대에 같은 행을 건드린 다른 트랜잭션이 있는가"를 검사한다. 이 검사가 Certification이다.

```
Node1: UPDATE users SET name='A' WHERE id=1  (seqno=100)
Node2: UPDATE users SET name='B' WHERE id=1  (seqno=100)

→ 같은 seqno 구간에서 같은 PK(id=1)를 수정
→ 충돌 감지
→ 나중에 커밋을 시도한 쪽 롤백
```

## 충돌 감지 기준

동일한 Global Sequence Number(seqno) 구간 내에서 동일한 PK를 수정한 트랜잭션이 둘 이상이면 충돌이다.

먼저 커밋에 성공한 트랜잭션이 이긴다. 나머지는 `wsrep_local_cert_failures` 카운터가 올라가고 롤백된다. 애플리케이션은 `deadlock` 에러를 받는다.

## CDC와의 충돌

롤백된 트랜잭션이 binlog에 기록됐다가 취소되는 타이밍이 문제다.

Galera는 Certification 결과가 나오기 전에 binlog에 먼저 기록하는 경우가 있다. CDC 도구가 이 타이밍에 binlog를 읽으면 실제로 적용되지 않은 변경을 이벤트로 만들어 Kafka에 발행할 수 있다.

이 문제를 피하려면 CDC 도구가 바라보는 노드를 하나로 고정하고, 해당 노드의 커밋 완료 이후 binlog만 읽도록 설정해야 한다.

## 낙관적 vs 비관적

비관적 방식은 충돌 가능성이 있으면 미리 잠근다. Galera의 낙관적 방식은 충돌이 드물다는 가정 하에 일단 허용하고 나중에 처리한다.

쓰기가 특정 행에 집중되지 않는 워크로드에서는 충돌이 적어 낙관적 방식이 유리하다. 반대로 같은 행에 쓰기가 몰리면 롤백이 잦아져 성능이 떨어진다.

참고: galera-cluster.md
참고: wsrep.md
참고: cdc.md
