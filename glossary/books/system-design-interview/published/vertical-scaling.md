# vertical-scaling (수직 확장)

Scale Up이라고도 부른다. 기존 서버에 CPU, RAM, DISK 같은 하드웨어 자원을 추가해서 성능을 높이는 방식.

## 장점

트래픽이 적을 때 단순하다는 게 가장 큰 장점이다. 코드나 아키텍처를 바꾸지 않아도 된다.

## 한계

두 가지 근본적인 한계가 있다.

1. 하드웨어 상한선이 존재한다 — 한 서버에 무한정 CPU와 메모리를 추가할 수 없다
2. Failover와 이중화(Redundancy)가 없다 — 서버 하나가 죽으면 서비스 전체가 다운된다

이 때문에 대규모 애플리케이션에서는 수직 확장보다 수평 확장이 더 적합하다.

## 데이터베이스의 수직 확장

Amazon RDS 기준으로 24TB RAM을 가진 DB 서버도 존재한다. Stack Overflow는 2013년 월 1천만 명을 단일 Master DB로 처리했다. 그러나 비용이 매우 높고 단일 장애점(SPOF) 문제가 남는다.

참고: horizontal-scaling.md, database-sharding.md
