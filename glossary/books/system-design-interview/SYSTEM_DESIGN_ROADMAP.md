# System Design Interview: An Insider's Guide 진도표
참고: System Design Interview: An Insider's Guide — Alex Xu (Second Edition)

목표: 각 챕터를 읽기 전에 스스로 먼저 설계해보고 책과 비교한다.
챕터 완료 기준: 읽기 완료 + 핵심 키워드 glossary 작성 + 직접 설계 재현 가능

---

## Part 1 — 기초 개념 및 프레임워크

### Chapter 1 — Scale from Zero to Millions of Users

- [x] single-server-setup.md — 단일 서버 구성, DNS·요청 흐름·트래픽 소스
- [x] vertical-scaling.md — Scale Up: 단일 서버에 CPU/RAM 추가, 한계와 단점
- [x] horizontal-scaling.md — Scale Out: 서버 수 증가, 로드 밸런서와 조합
- [x] load-balancer.md — 트래픽을 여러 웹 서버에 분산, private IP 사용 이유
- [x] database-replication.md — Master-Slave 구조, 읽기/쓰기 분리, 장애 처리
- [x] cache.md — 캐시 계층, read-through 전략, 만료·일관성·SPOF 고려사항
- [x] cdn.md — 정적 콘텐츠 배포, TTL·비용·캐시 무효화 고려사항
- [x] stateless-architecture.md — 세션 데이터를 외부 공유 저장소로 분리하는 방식
- [x] data-center.md — 멀티 데이터센터, GeoDNS 라우팅, 장애 시 트래픽 전환
- [x] message-queue.md — Producer-Consumer 비동기 처리, 컴포넌트 분리(decoupling)
- [x] database-sharding.md — 수평 파티셔닝, 샤드 키 선택, Resharding·Celebrity·Join 문제
- [x] ch01-scale-from-zero.md — Chapter 1 종합 복습 문서

### Chapter 2 — Back-of-the-Envelope Estimation

- [x] estimation-basics.md — 봉투 뒷면 계산의 기본 단위와 사고 방식
- [x] qps-estimation.md — QPS(Queries Per Second) 추정 방법
- [x] storage-estimation.md — 스토리지 용량 추정 방법
- [x] ch02-estimation.md — Chapter 2 종합 복습 문서

### Chapter 3 — A Framework for System Design Interviews

- [ ] interview-framework.md — 시스템 설계 인터뷰 4단계 프레임워크
- [ ] requirements-clarification.md — 기능 요구사항과 비기능 요구사항 도출 방법
- [ ] ch03-framework.md — Chapter 3 종합 복습 문서

---

## Part 2 — 핵심 컴포넌트 설계

### Chapter 4 — Design a Rate Limiter

- [ ] rate-limiter.md — Rate Limiter의 목적과 위치 (클라이언트 vs 서버 사이드)
- [ ] rate-limiting-algorithm.md — Token Bucket, Leaking Bucket, Fixed Window, Sliding Window 알고리즘 비교
- [ ] rate-limiter-distributed.md — 분산 환경에서의 Rate Limiter 구현 문제
- [ ] ch04-rate-limiter.md — Chapter 4 종합 복습 문서

### Chapter 5 — Design Consistent Hashing

- [ ] consistent-hashing.md — 일관된 해싱의 개념, 일반 해싱과의 차이
- [ ] virtual-node.md — 가상 노드(Virtual Node)로 데이터 분포 균형 맞추기
- [ ] ch05-consistent-hashing.md — Chapter 5 종합 복습 문서

### Chapter 6 — Design a Key-Value Store

- [ ] key-value-store.md — Key-Value 스토어 설계 요구사항과 트레이드오프
- [ ] cap-theorem.md — CAP 정리: Consistency, Availability, Partition Tolerance
- [ ] data-replication-strategy.md — 데이터 복제 전략, 쿼럼(Quorum) 합의
- [ ] gossip-protocol.md — Gossip Protocol로 노드 상태 전파하는 방식
- [ ] merkle-tree.md — Merkle Tree로 데이터 불일치 감지하는 방법
- [ ] ch06-key-value-store.md — Chapter 6 종합 복습 문서

### Chapter 7 — Design a Unique ID Generator in Distributed Systems

- [ ] distributed-id.md — 분산 환경에서 유일한 ID를 생성하는 방법들
- [ ] snowflake-id.md — Twitter Snowflake: 64비트 ID 구조와 생성 방식
- [ ] ch07-unique-id-generator.md — Chapter 7 종합 복습 문서

---

## Part 3 — 실전 시스템 설계

### Chapter 8 — Design a URL Shortener

- [ ] url-shortener.md — URL 단축 서비스 설계, 해시 함수 선택 기준
- [ ] redirect-type.md — 301 vs 302 리다이렉트의 차이와 선택 이유
- [ ] ch08-url-shortener.md — Chapter 8 종합 복습 문서

### Chapter 9 — Design a Web Crawler

- [ ] web-crawler.md — 웹 크롤러 설계, BFS 기반 URL 탐색 구조
- [ ] politeness-policy.md — 크롤링 속도 제한과 robots.txt 준수 정책
- [ ] url-frontier.md — URL Frontier: 크롤링 대기열 관리 방식
- [ ] ch09-web-crawler.md — Chapter 9 종합 복습 문서

### Chapter 10 — Design a Notification System

- [ ] notification-system.md — 알림 시스템 설계, Push/SMS/Email 채널 분리
- [ ] notification-reliability.md — 알림 전송 신뢰성 보장, 재시도와 중복 방지
- [ ] ch10-notification-system.md — Chapter 10 종합 복습 문서

### Chapter 11 — Design a News Feed System

- [ ] news-feed.md — 뉴스 피드 시스템 설계, Fanout 방식 비교
- [ ] fanout-on-write.md — Fanout on Write(Push 모델): 쓸 때 모든 팔로워에게 전파
- [ ] fanout-on-read.md — Fanout on Read(Pull 모델): 읽을 때 피드를 동적으로 생성
- [ ] ch11-news-feed.md — Chapter 11 종합 복습 문서

### Chapter 12 — Design a Chat System

- [ ] chat-system.md — 채팅 시스템 설계, 1:1 채팅과 그룹 채팅 차이
- [ ] websocket.md — WebSocket으로 서버-클라이언트 양방향 통신 유지
- [ ] chat-storage.md — 채팅 메시지 저장소 선택 기준, Key-Value vs RDBMS
- [ ] ch12-chat-system.md — Chapter 12 종합 복습 문서

### Chapter 13 — Design a Search Autocomplete System

- [ ] search-autocomplete.md — 검색 자동완성 시스템 설계, Trie 자료구조 활용
- [ ] trie.md — Trie 자료구조, 접두사 기반 검색의 시간복잡도
- [ ] ch13-search-autocomplete.md — Chapter 13 종합 복습 문서

### Chapter 14 — Design YouTube

- [ ] video-streaming.md — 비디오 스트리밍 시스템 설계, 인코딩 파이프라인
- [ ] dag-pipeline.md — DAG(Directed Acyclic Graph) 기반 비디오 처리 파이프라인
- [ ] ch14-youtube.md — Chapter 14 종합 복습 문서

### Chapter 15 — Design Google Drive

- [ ] google-drive.md — 클라우드 스토리지 설계, 파일 업로드/동기화 흐름
- [ ] block-storage.md — 파일을 블록 단위로 저장하는 방식, 델타 동기화
- [ ] ch15-google-drive.md — Chapter 15 종합 복습 문서

---

진행 방식: 챕터 읽기 전에 스스로 먼저 설계 시도 → 책과 비교 → 키워드 glossary 작성 → 종합 복습 문서 작성.
완료된 항목은 [x]로 표시.
