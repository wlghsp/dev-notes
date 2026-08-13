# Applied Algorithm Engineer 준비 로드맵 (미리캔버스 기준)
작성: 2026-08-13
전제: miridi_applied_algorithm_engineer.md에 정리된 JD를 기준으로 역량 갭을 좁힌다.

---

## JD를 세 갈래로 쪼개면

공고 전체가 한 문장으로 수렴한다 — "AI가 생성한 결과를 제품에 안전하게 태울지 판단하는 로직을 계산 가능한 구조로 만드는 일". 이걸 실제로 학습 가능한 단위로 쪼개면 세 갈래다.

1. 자료구조/알고리즘 응용 — tree/graph matching, bipartite matching, assignment problem, hash/indexing, candidate pruning
2. AI Agent 신뢰성 엔지니어링 — tool calling harness, precondition/postcondition/invariant, linter/validation, fallback policy
3. 랭킹/검색 시스템 — scoring function, rule-based evaluation, reranking

세 갈래 중 아무거나 하나만 잘해서는 부족하다. JD의 "우대 사항"이 이 세 개를 동시에 나열하고 있고, 1차 인터뷰가 "콘텐츠 구조와 디자인 구조를 매칭하는 라이브 과제"라는 점에서 1번이 가장 직접적인 평가 대상이다. 그래서 로드맵도 1번을 축으로 하고 2, 3번을 거기에 연결하는 순서로 짠다.

---

## Phase 1 — Matching 알고리즘 기본기

목표: "두 콘텐츠 구조 사이의 대응 관계를 찾는다"는 문제를 그래프/트리 알고리즘으로 표현할 수 있다.
완료 기준: bipartite matching과 tree edit distance를 직접 구현하고, 왜 이 문제에 이 알고리즘을 쓰는지 설명할 수 있다.

집중할 주제:

1. Tree 구조 문제
   Tree isomorphism, tree edit distance, subtree matching. 콘텐츠(디자인 요소 트리)를 비교하는 문제의 원형이 이거다.

2. Graph matching
   Bipartite matching (Hungarian algorithm), maximum matching. "원본 요소 집합 A와 대체 요소 집합 B를 1:1로 짝짓는다"는 문제가 정확히 이 형태다.

3. Assignment problem / constraint solving
   비용 함수가 있는 매칭(assignment problem), 제약 조건이 있는 매칭(constraint satisfaction)까지 확장.

학습 방법: 알고리즘 문제 풀이 사이트에서 tree/graph 카테고리를 골라 풀되, "이 문제가 실제 제품에서는 어떤 시나리오였을까"를 항상 한 줄로 적어둔다. leetcode572_Subtree_of_Another_Tree, leetcode173_Binary_Search_Tree_Iterator 같은 기존 풀이가 이미 트리 구조를 다루고 있으니 그 감각을 이어간다.

완료 기준 재확인: "디자인 요소 트리 A, B가 주어졌을 때 대응 가능한 노드 쌍을 찾아라" 유형 문제를 스스로 만들어서 30분 안에 구현 가능한 수준.

---

## Phase 2 — 대규모 후보군 탐색 (Indexing/Pruning)

목표: 후보가 수천~수만 개일 때 "빠르게 좁히는" 로직을 설계할 수 있다.
완료 기준: signature 기반 후보 축소 → 정밀 매칭이라는 2단계 파이프라인을 직접 짜본다.

집중할 주제:

1. Hash 기반 signature
   콘텐츠(색상, 레이아웃, 텍스트 구조 등)를 hash로 요약해서 "완전히 다른 후보"를 O(1)에 걸러내는 기법. LSH(Locality-Sensitive Hashing) 개념까지 포함.

2. Indexing 구조
   Inverted index, spatial index(트리/디자인 요소의 위치 기반 탐색이라면). "전체 스캔 대신 인덱스로 후보를 좁힌다"는 감각.

3. Deduplication / candidate pruning
   유사도 임계치로 중복 제거, 상위 K개만 남기는 pruning 전략.

참고: mysql-internals(glossary/books/mysql-internals)에서 인덱스가 탐색 범위를 줄이는 원리를 이미 DB 관점에서 다뤘다 — 같은 원리가 검색 후보 축소에도 그대로 적용된다는 걸 연결해서 이해하면 빠르다.

완료 기준 재확인: "후보 10만 개 중 유사한 것 top 100개를 1초 안에 뽑아라" 같은 문제를 시그니처+인덱스 조합으로 설계할 수 있다.

---

## Phase 3 — AI Agent Harness와 Validation

목표: "AI가 만든 결과를 제품에 써도 되는가"를 판단하는 코드를 설계할 수 있다.
완료 기준: precondition/postcondition/invariant를 명시한 tool harness와, 결과를 걸러내는 linter/validation rule을 하나씩 구현한다.

집중할 주제:

1. Tool calling harness 설계
   AI Agent가 tool을 호출하기 전(precondition)과 호출한 후(postcondition)에 무엇을 검증해야 하는지 구조화. invariant(항상 참이어야 하는 조건)를 코드로 표현하는 법.

2. Linter / validation rule
   "잘못된 입력, 잘못된 실행 순서, 낮은 품질 결과"를 사전에 걸러내는 규칙 엔진. rule engine 패턴(if-then 규칙의 조합, 우선순위, 충돌 처리).

3. Fallback policy
   실패 케이스를 수집해서 "허용 가능한 결과 vs 거절해야 하는 결과"를 구분하는 정책 설계. 단순 try-catch가 아니라, 품질 기준에 따른 단계적 fallback(예: 최선 결과 실패 시 차선책 자동 전환).

4. LLM 출력 검증
   Hallucination 방지, 출력 스키마 검증, 품질 평가 로직. 이 레포 glossary/topics/ai-llm에 관련 개념이 쌓여 있다면 우선 참고.

완료 기준 재확인: "AI가 생성한 결과 100개 중 제품에 바로 쓸 수 없는 것을 자동으로 걸러내는 검증 파이프라인"을 설계 문서 수준으로 작성할 수 있다.

---

## Phase 4 — Ranking / Scoring 시스템

목표: 여러 후보 중 "가장 나은 것"을 정하는 로직을 설계할 수 있다.
완료 기준: rule-based scoring과 reranking을 결합한 파이프라인을 하나 만들어본다.

집중할 주제:

1. Scoring function 설계
   여러 신호(유사도, 품질, 제약 조건 충족 여부)를 하나의 점수로 합치는 법. 가중치를 어떻게 정할지, 신호가 서로 상충할 때 어떻게 처리할지.

2. Rule-based evaluation
   ML 기반이 아니라 명시적 규칙으로 결과를 평가하는 방식. JD가 "rule-based evaluation"을 명시하고 있어서, 이 포지션은 ML보다 명시적 로직 설계에 가깝다는 신호로 읽힌다.

3. Reranking
   1차로 빠르게 후보를 좁힌 뒤(Phase 2의 pruning), 2차로 정밀하게 재정렬하는 2단계 구조.

완료 기준 재확인: "후보 100개를 스코어링 규칙으로 정렬하고, 왜 이 순서인지 설명 가능한" 파이프라인.

---

## 실전 연결 — PoC 반복 훈련

JD가 "완성도보다 학습 속도를 우선해 실험을 반복해본 경험"을 우대사항으로 명시한다. 위 네 Phase를 각각 이론으로만 끝내지 말고, 작은 PoC로 즉시 구현해보는 게 핵심이다.

제안하는 연습 사이클:
1. 문제를 하나 정한다 (예: "이미지 두 장의 레이아웃 구조를 비교해서 유사도를 매긴다")
2. 1시간 안에 가장 단순한 버전으로 구현한다 (완성도 무시)
3. 결과를 보고 실패 케이스를 3개 이상 찾는다
4. 그 실패 케이스를 fallback/validation 로직으로 보완한다

이 사이클 자체가 1차 인터뷰의 "라이브 과제"와 거의 동일한 형태다.

---

## 타임라인 (참고용)

Phase 1 — 3~4주 (tree/graph matching 알고리즘 집중)
Phase 2 — 2~3주 (indexing/pruning)
Phase 3 — 3~4주 (AI harness/validation — 실무 경험이 적은 영역이라 가장 오래 걸릴 수 있음)
Phase 4 — 2주 (ranking/scoring)
실전 PoC 연습 — Phase 1 끝난 시점부터 병행 시작

총 2~3개월. 기존 ProblemSolving_Everyday 레포에서 알고리즘 문제를 이미 꾸준히 풀고 있다면 Phase 1은 단축 가능.

---

## 현재 위치

- 시작 전 — Phase 1부터 진입 예정
