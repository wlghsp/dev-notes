# dev-notes

CS 기초부터 내부 구조까지 깊게 이해하는 개발자로 가는 학습 기록.
블로그 초안, 트레이닝 시리즈, 용어집을 관리한다.

---

## 트레이닝

### JVM 트레이닝
> JVM 내부 구조를 깊게 파고, 블로그 + 발표로 실력을 증명한다.

- Phase 1 — JVM이란 무엇인가 | 실습 ✅ | 블로그 ⬜
- Phase 2 — Runtime Data Areas — 메모리 영역 해부 | 실습 ✅ | 블로그 ⬜
- Phase 3 — Garbage Collection | 실습 ⬜ | 블로그 ⬜
- Phase 4 — Class Loading | 실습 ⬜ | 블로그 ⬜
- Phase 5 — JIT 컴파일러 | 실습 ⬜ | 블로그 ⬜
- Phase 6 — 실전 트러블슈팅 | 실습 ⬜ | 블로그 ⬜

→ 로드맵 상세: [jvm-training/TRAINING_ROADMAP.md](jvm-training/TRAINING_ROADMAP.md)

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

→ 로드맵 상세: [db-internals/TRAINING_ROADMAP.md](db-internals/TRAINING_ROADMAP.md)

---

## 용어집

블로그 주제로 키우기엔 작지만, 흘려보내기엔 중요한 개념들을 단독 파일로 쌓는다.

→ [glossary/](glossary/)

---

## 블로그 발행 목록

- [Why Single Thread Event Loop](blog/published/why-single-thread-event-loop.md)
- [Why Single Thread Event Loop — 용어집](blog/published/why-single-thread-event-loop-glossary.md)
- [Servlet이란 무엇인가](blog/published/servlet-what-and-why.md)
- [Dispatcher Servlet 흐름](blog/published/dispatcher-servlet-flow.md)
- [Java vs Node.js 스레드 모델](blog/published/java-vs-nodejs-thread-model.md)
- [Multithreading — Node.js에서 Spring으로](blog/published/multithreading-from-nodejs-to-spring.md)
- [Thread Model — Node / Java / Spring](blog/published/thread-model-node-java-spring.md)
- [Prometheus — What and Why](blog/published/prometheus-what-and-why.md)
