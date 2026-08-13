# Applied Algorithm Engineer 트레이닝 로드맵
## "미리캔버스 JD 기준으로 알고리즘/AI Agent 설계 역량 키우기"

> 문서를 만드는 게 목표가 아니다. 설명할 수 있는 수준, 그리고 직접 짜서 검증 가능한 수준이 목표다.
> JD 원문은 career/miridi_applied_algorithm_engineer.md, 전체 배경은 career/miridi_applied_algorithm_engineer_roadmap.md 참고.

---

## 진행 원칙

각 Phase는 이론 정리 → PoC 구현 → 실패 케이스 분석 순서를 지킨다.
"문제 유형을 안다"는 완료가 아니다 — 직접 구현하고, 왜 이 알고리즘을 골랐는지 설명할 수 있어야 완료다.
Phase 3(AI Agent Harness)은 실무 경험이 가장 적은 영역이므로 시간을 넉넉히 잡는다.

---

## Phase 현황

- Phase 1 — Tree/Graph Matching 알고리즘 — 시작 전
- Phase 2 — Indexing / Candidate Pruning — 시작 전
- Phase 3 — AI Agent Harness / Validation — 시작 전
- Phase 4 — Ranking / Scoring 시스템 — 시작 전
- 실전 PoC 연습 — Phase 1 종료 후 병행 시작

---

## 각 Phase 상세

### Phase 1: Tree/Graph Matching 알고리즘
**파일**: `matching-algorithms.md` (미작성)
**완료 기준**: bipartite matching과 tree edit distance를 직접 구현하고, "디자인 요소 트리 A, B 사이 대응 노드 쌍을 찾아라" 유형을 30분 안에 풀 수 있다.
**다룰 개념**: tree isomorphism, tree edit distance, bipartite matching(Hungarian algorithm), assignment problem, constraint solving

### Phase 2: Indexing / Candidate Pruning
**파일**: `indexing-pruning.md` (미작성)
**완료 기준**: signature 기반 후보 축소 → 정밀 매칭이라는 2단계 파이프라인을 직접 설계한다.
**다룰 개념**: hash 기반 signature, LSH, inverted index, deduplication, candidate pruning
**연결**: glossary/books/mysql-internals의 인덱스 개념을 검색 후보 축소 관점으로 재해석

### Phase 3: AI Agent Harness / Validation
**파일**: `ai-agent-harness.md` (미작성)
**완료 기준**: precondition/postcondition/invariant를 명시한 tool harness와, 결과를 걸러내는 validation rule을 각각 하나씩 구현한다.
**다룰 개념**: tool calling harness, rule engine, fallback policy, LLM 출력 검증(hallucination 방지, 스키마 검증)

### Phase 4: Ranking / Scoring 시스템
**파일**: `ranking-scoring.md` (미작성)
**완료 기준**: rule-based scoring과 reranking을 결합한 파이프라인을 하나 만든다.
**다룰 개념**: scoring function 설계, rule-based evaluation, 2단계 reranking 구조

---

## 실전 PoC 연습 사이클

Phase 1이 끝나는 시점부터 아래 사이클을 병행한다. 1차 인터뷰가 "라이브 과제" 형태이므로 이 사이클 자체가 연습이다.

1. 문제를 하나 정한다 (예: 이미지 두 장의 레이아웃 구조를 비교해서 유사도를 매긴다)
2. 1시간 안에 가장 단순한 버전으로 구현한다 (완성도 무시)
3. 결과를 보고 실패 케이스를 3개 이상 찾는다
4. 그 실패 케이스를 fallback/validation 로직으로 보완한다

---

## 지금 당장 할 것

**Phase 1 시작** → `matching-algorithms.md` 생성부터
- tree edit distance, bipartite matching 개념 정리
- 알고리즘 문제로 감 잡기 (ProblemSolving_Everyday 레포의 tree 관련 풀이와 연결)
- 직접 구현 → 실패 케이스 정리 → 다음 Phase
