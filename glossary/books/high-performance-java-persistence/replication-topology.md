# replication-topology (복제 토폴로지)

데이터베이스 복제를 동기화 방식(동기/비동기)과 쓰기 권한 분산(Master-Slave/Multi-Master) 두 축으로 나눠서 본 것.

## 동기 복제 vs 비동기 복제

Slave를 동기(synchronous)로 둘지 비동기(asynchronous)로 둘지는 일관성과 응답 시간의 트레이드오프다.

동기 복제(hot standby) — Master가 쓰기를 커밋하기 전에 동기 Slave의 응답(ack)을 기다린다. Slave는 항상 Master의 정확한 사본이므로, Master 장애 시 데이터 손실 없이 즉시 새 Master로 승격할 수 있다. 대신 매 쓰기마다 대기 시간이 늘어 응답 시간이 길어진다. 보통 Slave 전부가 아니라 하나만 동기로 두는 방식이 일반적이다.

비동기 복제(warm standby) — Master는 Slave의 응답을 기다리지 않고 커밋한다. 응답 시간은 짧지만 Slave가 Master보다 뒤처질 수 있다(eventually consistent). Master 장애 시 승격되는 Slave가 최신 데이터를 갖고 있지 않을 수 있어, 일관성과 지속성을 응답 시간·처리량과 맞바꾼 것이다.

복제는 binary 방식(Master의 WAL을 그대로 전달)과 statement-based 방식(Slave에서 동일한 SQL 문을 재실행)으로 나뉜다.

## Multi-Master 복제

Master-Slave 구조에서는 쓰기가 Master 한 대로 몰려 병목이 될 수 있다. Multi-Master 복제는 모든 노드가 동등하게 읽기/쓰기를 모두 받을 수 있게 해서 쓰기 처리량 자체를 분산시킨다.

문제는 단일 진실 공급원(source of truth)이 사라진다는 점이다. 같은 데이터가 서로 다른 노드에서 동시에 수정되면 충돌이 발생할 수 있다. 해결 방식은 두 가지다.

- 충돌 회피 — 2단계 커밋(Two-Phase Commit)으로 모든 노드를 하나의 분산 트랜잭션에 묶어 항상 동기화 상태를 유지한다. 데이터 일관성은 보장되지만 노드 간 네트워크(특히 WAN)로 인해 쓰기 응답 시간이 크게 늘어난다.
- 충돌 감지 및 해결 — 비동기로 복제하고, 충돌이 감지되면 자동 병합 알고리즘으로 해결을 시도한다. 자동 병합이 실패하면 수동 개입이 필요하다. 응답 시간은 짧지만 충돌 해결 로직이 복잡해진다.

참고: usl.md(복제로 늘어난 노드 수가 실제로 처리량에 어떻게 반영되는지의 이론적 배경). 일반적인 Master-Slave 개념 자체는 database-replication.md(system-design-interview) 참고.
