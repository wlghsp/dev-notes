# stateless-architecture (무상태 아키텍처)

웹 서버가 세션 데이터 같은 상태(State)를 자신에게 저장하지 않고 외부 공유 저장소에 두는 아키텍처. 웹 티어의 수평 확장을 가능하게 하는 핵심 조건이다.

## Stateful vs Stateless

Stateful 서버는 클라이언트의 상태를 요청 간에 기억한다. User A의 세션 데이터가 Server 1에 저장되어 있으면 User A의 모든 요청은 반드시 Server 1로 가야 한다. 이를 sticky session이라 한다. 로드 밸런서가 sticky session을 지원하더라도 서버 추가/제거가 어렵고 서버 장애 처리가 복잡해진다.

Stateless 서버는 상태를 저장하지 않는다. 어떤 서버로 요청이 가도 같은 결과를 반환할 수 있다.

> 📷 Figure 1-12 (책 p.19) — Stateful 아키텍처: User별로 고정 서버 필요
> 📷 Figure 1-13 (책 p.20) — Stateless 아키텍처: 모든 서버가 공유 저장소에서 상태를 읽음

## 구현 방법

세션 데이터를 웹 서버 밖의 공유 데이터 저장소(Relational DB, Redis, NoSQL 등)에 저장한다. 모든 웹 서버가 이 저장소에서 상태를 읽는다.

## 효과

상태를 서버에서 분리하면 Auto Scaling이 쉬워진다. 트래픽에 따라 서버를 자동으로 추가하거나 제거할 수 있다.

> 📷 Figure 1-14 (책 p.21) — Stateless + Auto Scale 구성도 (Server 1~4, NoSQL 공유)

참고: load-balancer.md, horizontal-scaling.md, data-center.md
