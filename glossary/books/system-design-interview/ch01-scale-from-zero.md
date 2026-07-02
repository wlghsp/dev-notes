# Chapter 1 — Scale from Zero to Millions of Users

키워드 파일: single-server-setup.md, vertical-scaling.md, horizontal-scaling.md, load-balancer.md, database-replication.md, cache.md, cdn.md, stateless-architecture.md, data-center.md, message-queue.md, database-sharding.md

---

## 이 챕터가 답하는 질문

사용자 한 명을 감당하는 단일 서버에서 출발해, 수백만 사용자를 감당하는 시스템까지 어떤 순서로, 어떤 이유로 컴포넌트를 하나씩 추가해 나가는가.

---

## 출발점: 단일 서버

모든 것이 서버 한 대에 있다(single-server-setup.md). 사용자가 도메인으로 접근하면 DNS가 IP를 반환하고, 브라우저/모바일 앱이 그 IP로 직접 HTTP 요청을 보낸다. 웹 서버가 HTML 또는 JSON을 반환한다. 트래픽이 늘어나기 전까지는 이 구조로 충분하다.

## 첫 번째 분리: 웹 서버와 DB

사용자가 늘면 웹 트래픽과 데이터베이스를 독립적으로 확장할 수 있어야 한다. 서버를 웹 티어와 데이터 티어로 분리하는 게 수평 확장의 첫걸음이다.

## 확장의 두 가지 축: 수직 vs 수평

수직 확장(vertical-scaling.md)은 서버 한 대에 CPU/RAM을 더 붙이는 방식이다. 단순하지만 하드웨어 상한선이 있고 Failover가 없다는 근본적 한계가 있다.

수평 확장(horizontal-scaling.md)은 서버를 더 추가하는 방식이다. 대규모 애플리케이션에서는 수직 확장의 한계를 피하기 위해 결국 수평 확장으로 간다. 단, 수평 확장을 하려면 트래픽을 여러 서버에 나눠줄 무언가가 필요해진다 — 그게 로드 밸런서다.

## 로드 밸런서로 웹 티어의 Failover 해결

로드 밸런서(load-balancer.md)는 Public IP로 트래픽을 받아 Private IP로 통신하는 여러 웹 서버에 분산한다. 서버 하나가 죽어도 나머지가 트래픽을 받고, 트래픽이 늘면 서버를 추가하기만 하면 된다. 이로써 웹 티어의 이중화 문제가 해결된다.

## 데이터 티어의 이중화: 복제

웹 티어와 마찬가지로 데이터 티어도 단일 장애점을 없애야 한다. 데이터베이스 복제(database-replication.md)는 Master-Slave 구조로 쓰기는 Master에, 읽기는 여러 Slave에 분산한다. 읽기 비율이 쓰기보다 훨씬 높은 일반적인 워크로드에 잘 맞는다.

## 부하를 줄이는 두 가지: 캐시와 CDN

DB 앞에 캐시(cache.md)를 두면 자주 조회되는 데이터를 메모리에서 바로 응답해 DB 부하를 줄인다. Read-Through 전략, 만료 정책, 캐시-DB 일관성, SPOF, Eviction 정책이 핵심 고려사항이다.

정적 콘텐츠(이미지, CSS, JS)는 CDN(cdn.md)으로 옮겨서 사용자와 지리적으로 가까운 서버에서 서빙한다. 캐시는 "DB 앞"에서 동적 데이터를, CDN은 "인터넷 경로상"에서 정적 자산을 줄여준다는 차이가 있다.

## 웹 티어를 완전히 수평 확장하려면: Stateless

로드 밸런서와 여러 웹 서버가 있어도, 세션 데이터가 서버 하나에만 있으면(Stateful) 그 사용자의 모든 요청이 같은 서버로 가야 한다(sticky session). 이러면 서버 추가/제거가 어려워진다.

무상태 아키텍처(stateless-architecture.md)는 세션 데이터를 공유 저장소(Redis, NoSQL 등)로 빼서 어떤 서버가 요청을 받아도 동일하게 처리할 수 있게 만든다. 이게 되어야 비로소 Auto Scaling이 가능해진다.

## 지리적 확장: 멀티 데이터센터

사용자가 전 세계로 퍼지면 데이터센터도 여러 지역에 둔다(data-center.md). GeoDNS가 사용자를 가장 가까운 DC로 라우팅하고, 한 DC가 죽으면 트래픽을 다른 DC로 전환한다. 이때 트래픽 리다이렉션, 데이터 동기화, 배포 일관성이라는 새로운 과제가 생긴다.

## 컴포넌트 분리: 메시지 큐

시스템이 커지면 컴포넌트 간 강한 결합이 병목이 된다. 메시지 큐(message-queue.md)는 Producer와 Consumer를 분리해서 서로 독립적으로 확장 가능하게 만든다. Consumer가 잠깐 다운돼도 Producer는 계속 메시지를 큐에 쌓을 수 있다.

이 시점부터 Logging, Metrics, Automation(CI/CD)도 선택이 아니라 필수가 된다.

## 데이터 티어의 마지막 관문: 샤딩

복제만으로는 데이터 "양" 자체가 감당 안 되는 시점이 온다. 샤딩(database-sharding.md)은 DB를 여러 샤드로 수평 분할한다. 샤드 키 선택이 핵심이며, Resharding·Celebrity Problem(핫스팟)·Join 제약이라는 세 가지 새로운 복잡성이 따라온다.

## 전체 흐름 요약

1. 단일 서버 → 웹/DB 분리 → 로드 밸런서(웹 티어 이중화) → DB 복제(데이터 티어 이중화)
2. 캐시 + CDN으로 부하 감소 → Stateless로 웹 티어 완전 수평 확장
3. 멀티 데이터센터로 지리적 확장 → 메시지 큐로 컴포넌트 분리
4. 샤딩으로 데이터 티어 수평 확장

각 단계는 이전 단계의 병목을 해결하기 위해 추가된다는 점이 이 챕터 전체를 관통하는 논리다.

> 📷 Figure 1-23 (책 p.31) — 로드 밸런서, 샤딩된 DB, 캐시, 메시지 큐, NoSQL, 모니터링 도구가 모두 결합된 최종 아키텍처
