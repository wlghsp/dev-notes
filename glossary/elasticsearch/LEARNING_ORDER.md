# Elasticsearch 학습 순서

개념이 쌓이는 방향으로 구성했다. 앞 개념을 이해해야 뒤 개념이 자연스럽게 들어온다.

---

## 1단계 — 데이터 모델 이해

Elasticsearch가 데이터를 어떻게 표현하는지부터 잡는다.

1. document.md — 기본 저장 단위
2. field.md — document를 구성하는 키-값, text vs keyword 구분
3. mapping.md — 필드 타입을 정의하는 스키마
4. index.md — document들을 담는 논리적 단위

---

## 2단계 — 검색의 핵심 원리

왜 빠르게 찾는지, 어떻게 관련성을 판단하는지.

5. inverted-index.md — 역색인: 검색 엔진의 핵심 자료구조
6. analyzer.md — 텍스트를 토큰으로 변환하는 파이프라인
7. full-text-search.md — 전문 검색 개념과 동작
8. relevance-score.md — 관련성 점수 계산 원리
9. bm25.md — 기본 점수 알고리즘 (TF-IDF 개선)

---

## 3단계 — 물리적 저장 구조

데이터가 실제로 어떻게 저장되고 관리되는지.

10. segment.md — Lucene의 물리적 저장 단위, 불변성
11. refresh.md — buffer → Segment, 검색 가능 상태로 전환
12. flush.md — Segment → 디스크, 영구 저장
13. translog.md — WAL, 장애 복구 안전망
14. merge.md — 작은 Segment를 합쳐 최적화
15. near-real-time.md — 왜 1초 지연이 생기는가

---

## 4단계 — 분산 구조

클러스터로 확장하는 방법과 원리.

16. shard.md — 인덱스를 물리적으로 나누는 단위
17. replica.md — 복제본, 가용성과 읽기 성능
18. node.md — Master / Data / Coordinating 역할 분리
19. cluster.md — 여러 노드가 모인 전체 구조, Green/Yellow/Red

---

## 5단계 — 운영과 쿼리

실제로 쓰고 관리하는 방법.

20. query-dsl.md — JSON으로 쿼리 작성, bool / match / term / range
21. shard-allocation.md — Shard가 노드에 배치되는 원리
22. rebalancing.md — 노드 변화 시 Shard 재분배
