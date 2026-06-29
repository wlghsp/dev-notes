# database-replication (데이터베이스 복제)

원본(Master)과 사본(Slave) 사이의 Master-Slave 관계로 데이터를 복제하는 기법.

## 역할 분리

Master DB는 쓰기 연산만 담당한다. INSERT, DELETE, UPDATE는 반드시 Master로 보낸다. Slave DB는 Master로부터 데이터를 복사해서 읽기 연산만 처리한다.

대부분의 애플리케이션은 읽기 비율이 쓰기보다 훨씬 높다. 그래서 Slave DB 수가 Master보다 많은 게 일반적이다.

> 📷 Figure 1-5 (책 p.9) — Master 1개, Slave 3개 복제 구성도

## 장점

1. 성능 — 읽기 쿼리가 여러 Slave로 분산되어 병렬 처리 가능
2. 신뢰성 — DB 서버가 재해로 파괴되어도 다른 위치에 데이터가 보존됨
3. 고가용성 — 한 DB가 오프라인이어도 다른 DB에서 데이터에 접근 가능

## 장애 시나리오

Slave가 한 대만 남고 오프라인이 되면 — 읽기 요청을 Master로 임시 전환, 새 Slave를 즉시 추가한다.

Master가 오프라인이 되면 — Slave 하나를 새 Master로 승격한다. 그러나 Slave 데이터가 최신이 아닐 수 있어 복구 스크립트로 누락 데이터를 채워야 한다. Multi-master나 Circular 복제로 해결할 수도 있지만 더 복잡하다.

> 📷 Figure 1-6 (책 p.11) — 로드 밸런서 + 복제 구성 전체 흐름도

참고: load-balancer.md, database-sharding.md
