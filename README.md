# dev-notes

CS 기초부터 내부 구조까지 깊게 이해하는 개발자로 가는 학습 기록.
블로그 초안, 트레이닝 시리즈, 용어집, 발표 준비 자료를 관리한다.

---

## 트레이닝

### JVM 트레이닝
> JVM 내부 구조를 깊게 파고, 블로그 + 발표로 실력을 증명한다.

- Phase 0 — 컴퓨터 메모리 구조 (선행지식) | 실습 ⬜ | 블로그 ⬜
- Phase 1 — JVM이란 무엇인가 | 실습 ✅ | 블로그 ⬜
- Phase 2 — Runtime Data Areas — 메모리 영역 해부 | 실습 ✅ | 블로그 ⬜
- Phase 3 — Garbage Collection | 실습 ⬜ | 블로그 ⬜
- Phase 4 — Class Loading | 실습 ⬜ | 블로그 ⬜
- Phase 5 — JIT 컴파일러 | 실습 ⬜ | 블로그 ⬜
- Phase 6 — 실전 트러블슈팅 | 실습 ⬜ | 블로그 ⬜

로드맵 상세: jvm-training/TRAINING_ROADMAP.md

---

### DB Internals 트레이닝
> DB 내부 구조를 이해하고, 실행 계획과 성능 문제를 스스로 판단할 수 있는 수준으로.

- Phase 0 — 스토리지 기초 — Page, Buffer Pool, I/O | 실습 ✅ | 블로그 ✅
- Phase 1 — DBMS 아키텍처 — Storage Engine 컴포넌트 | 실습 ⬜ | 블로그 ✅
- Phase 2 — B-Tree 내부 구조 | 실습 ⬜ | 블로그 ⬜
- Phase 3 — LSM-Tree — 쓰기 최적화 구조 | 실습 ⬜ | 블로그 ⬜
- Phase 4 — 트랜잭션 & MVCC | 실습 ⬜ | 블로그 ⬜
- Phase 5 — WAL & Crash Recovery | 실습 ⬜ | 블로그 ⬜
- Phase 6 — 인덱스 내부 구조 & 쿼리 실행 엔진 | 실습 ⬜ | 블로그 ⬜
- Phase 7 — 실전 트러블슈팅 | 실습 ⬜ | 블로그 ⬜

로드맵 상세: db-internals/TRAINING_ROADMAP.md

---

### Distributed Systems 트레이닝
> 구글시트, Kafka, etcd 같은 실존 시스템이 실제로 어떻게 만들어졌는지 사례로 파고든다.

로드맵 상세: distributed-systems-training/TRAINING_ROADMAP.md
학습 완료 자료: distributed-systems-training/studied/

---

### Network 트레이닝
> 네트워크를 AI 없이 디버깅할 수 있는 개발자가 되는 게 목표.

로드맵 상세: network-training/TRAINING_ROADMAP.md

---

## 용어집

블로그 주제로 키우기엔 작지만, 흘려보내기엔 중요한 개념들을 단독 파일로 쌓는다.

- glossary/topics/ — 주제별 개념 정리 (kafka, mysql, elasticsearch, haproxy, os-network, database, typescript, ai-llm 등)
- glossary/books/ — 기술 서적 챕터 단위 학습 (DDIA, Database Internals, MySQL Internals, Java Performance, CSAPP, Kubernetes in Action, Art of Multiprocessor, Algorithm Design Manual, Modern OS, Grokking Concurrency, System Design Interview, Build LLM from Scratch, Hands-On LLM 등)
- glossary/*/published/ — 지호님이 직접 블로그에 발행 완료한 파일. 발행 여부는 지호님이 직접 관리한다.

---

## 기타 학습/작업 기록

- sysadmin/ — 서버 운영, 인프라, AI 활용 관련 노트 및 발표 자료
- incident/ — 실제 장애 대응 기록
- interview/ — 기술 면접 대비 자료 (자료구조/알고리즘, OS, 네트워크, DB, Java/Spring 등)
- career/ — 커리어 로드맵
- spring_camp_2026/ — Spring Camp 2026 발표 준비 자료
- ai_efficiency_talk/ — AI 활용 효율성 관련 발표 준비 자료
- token_competition_game/ — 사이드 프로젝트 기획 노트
- spring/, java/ — Spring, Java 학습 노트

---

## 블로그

발행 전 원고는 blog/ 하위 주제별 폴더(cdc, database, network, inflearn 등)에, 발행 완료된 글은 blog/published/ 에 모은다.
