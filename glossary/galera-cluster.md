# Galera Cluster

MySQL/MariaDB에서 모든 노드에 읽기/쓰기가 가능한 Multi-Master 동기 복제 클러스터.

일반 MySQL 복제(Primary-Replica)는 Primary에만 쓰고 Replica는 읽기 전용이다. Galera는 모든 노드가 Primary처럼 동작한다.

## 핵심 특성

동기 복제다. 트랜잭션이 커밋되려면 클러스터의 모든 노드가 해당 트랜잭션을 받아야 한다. 노드 하나가 죽어도 데이터 유실이 없다.

단, "받았다"는 것이 "적용했다"는 의미는 아니다. Galera는 트랜잭션을 받고 충돌 여부를 검증한 뒤 적용한다. 이 검증 과정이 Certification-based Replication이다.

## Multi-Master의 문제

모든 노드에 쓸 수 있으니 같은 행을 동시에 여러 노드에서 수정할 수 있다. 이때 충돌이 발생한다. Galera는 이를 낙관적으로 처리한다 — 일단 쓰고, 나중에 충돌을 감지해서 한쪽을 롤백한다.

이 낙관적 충돌 감지가 CDC와 충돌한다. 롤백된 트랜잭션이 binlog에 잠깐 기록됐다가 무효화되는 타이밍에 CDC 도구가 읽으면 실제로 적용되지 않은 변경을 전파할 수 있다.

## CDC와의 관계

일반 MySQL은 Primary 하나의 binlog만 읽으면 된다. Galera는 어느 노드의 binlog를 읽을지, 노드 장애 시 다른 노드로 어떻게 전환할지, 노드마다 다른 binlog 순서를 어떻게 처리할지가 추가 문제가 된다.

참고: certification-based-replication.md
참고: wsrep.md
참고: galera-auto-increment.md
참고: cdc.md
