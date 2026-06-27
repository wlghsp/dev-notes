# Algorithm Design Manual Glossary 진도표
참고: The Algorithm Design Manual, 3rd Edition (2020) — Steven S. Skiena

---

## Chapter 1 — Introduction to Algorithm Design

- [x] algorithm.md — 알고리즘 정의, correctness·efficiency·ease of implementation
- [x] heuristic.md — 올바름을 보장하지 않는 절차, 알고리즘과의 차이
- [x] problem-instance.md — 문제 명세와 구체적 입력 인스턴스의 구분
- [x] correctness.md — 모든 입력에서 올바른 답 보장, 증명 방법
- [x] counterexample.md — 반례 찾기 전략, verifiability·simplicity
- [x] induction.md — 재귀적/증분적 알고리즘 올바름 증명, 재귀와의 관계
- [x] proof-by-contradiction.md — 모순에 의한 증명 구조
- [x] modeling.md — 실세계 문제를 추상 구조로 변환하는 과정
- [x] combinatorial-object.md — 7가지 핵심 추상 구조 (Permutations, Subsets, Trees, Graphs, Points, Polygons, Strings)
- [x] recursive-object.md — 큰 구조에서 원소 제거 시 같은 유형의 작은 구조가 되는 성질
- [x] ch01-introduction-to-algorithm-design.md — 챕터 종합 문서

## Chapter 2 — Algorithm Analysis

- [ ] ram-model.md — Random Access Machine 계산 모델
- [ ] big-oh.md — 점근적 복잡도 표기법
- [ ] best-worst-average-case.md — 최선·최악·평균 경우 복잡도
- [ ] dominance-relation.md — 함수 증가율 비교와 지배 관계
- [ ] logarithm.md — 로그의 성질과 알고리즘에서의 역할
- [ ] summation.md — 합산 기법

## Chapter 3 — Data Structures

- [ ] contiguous-vs-linked.md — 연속 자료구조 vs 연결 자료구조
- [ ] array.md — 배열
- [ ] linked-list.md — 연결 리스트
- [ ] stack.md — 스택
- [ ] queue.md — 큐
- [ ] dictionary.md — 딕셔너리 ADT
- [ ] binary-search-tree.md — 이진 탐색 트리
- [ ] balanced-search-tree.md — 균형 탐색 트리
- [ ] priority-queue.md — 우선순위 큐
- [ ] heap.md — 힙
- [ ] hash-table.md — 해시 테이블
- [ ] collision-resolution.md — 해시 충돌 해결
- [ ] canonicalization.md — 정규화

## Chapter 4 — Sorting

- [ ] heapsort.md — 힙 정렬
- [ ] mergesort.md — 병합 정렬
- [ ] quicksort.md — 퀵 정렬
- [ ] randomized-quicksort.md — 랜덤화 퀵 정렬
- [ ] distribution-sort.md — 분배 정렬
- [ ] lower-bound-sorting.md — 정렬의 하한
- [ ] sorting-applications.md — 정렬의 응용

## Chapter 5 — Divide and Conquer

- [ ] binary-search.md — 이진 탐색
- [ ] recurrence-relation.md — 점화식
- [ ] divide-and-conquer.md — 분할 정복 설계 기법
- [ ] fast-multiplication.md — 빠른 곱셈
- [ ] closest-pair.md — 최근접 점 쌍
- [ ] convolution.md — 합성곱

## Chapter 6 — Hashing and Randomized Algorithms

- [ ] probability-basics.md — 확률 기초
- [ ] balls-and-bins.md — 공과 상자 모델
- [ ] bloom-filter.md — 블룸 필터
- [ ] birthday-paradox.md — 생일 역설
- [ ] perfect-hashing.md — 완전 해싱
- [ ] minwise-hashing.md — 민와이즈 해싱
- [ ] string-matching-hash.md — 해시 기반 문자열 매칭
- [ ] primality-testing.md — 소수 판별
- [ ] random-number-generation.md — 난수 생성

## Chapter 7 — Graph Traversal

- [ ] graph.md — 그래프 정의, 종류
- [ ] graph-representation.md — 그래프 표현 방식
- [ ] adjacency-list.md — 인접 리스트
- [ ] adjacency-matrix.md — 인접 행렬
- [ ] bfs.md — 너비 우선 탐색
- [ ] dfs.md — 깊이 우선 탐색
- [ ] connected-components.md — 연결 요소
- [ ] bipartite.md — 이분 그래프
- [ ] topological-sort.md — 위상 정렬
- [ ] strongly-connected-components.md — 강연결 요소
- [ ] articulation-vertex.md — 단절점

## Chapter 8 — Weighted Graph Algorithms

- [ ] minimum-spanning-tree.md — 최소 신장 트리
- [ ] prim-algorithm.md — 프림 알고리즘
- [ ] kruskal-algorithm.md — 크루스칼 알고리즘
- [ ] union-find.md — 유니온-파인드 자료구조
- [ ] shortest-path.md — 최단 경로
- [ ] dijkstra-algorithm.md — 다익스트라 알고리즘
- [ ] all-pairs-shortest-path.md — 전체 쌍 최단 경로
- [ ] transitive-closure.md — 추이적 폐쇄
- [ ] network-flow.md — 네트워크 플로우
- [ ] bipartite-matching.md — 이분 매칭
- [ ] randomized-min-cut.md — 랜덤화 최소 컷

## Chapter 9 — Combinatorial Search

- [ ] backtracking.md — 백트래킹
- [ ] search-pruning.md — 탐색 가지치기
- [ ] subset-enumeration.md — 부분집합 열거
- [ ] permutation-enumeration.md — 순열 열거
- [ ] best-first-search.md — 최선 우선 탐색
- [ ] a-star.md — A* 휴리스틱

## Chapter 10 — Dynamic Programming

- [ ] memoization.md — 메모이제이션
- [ ] dynamic-programming.md — 동적 프로그래밍
- [ ] edit-distance.md — 편집 거리
- [ ] longest-increasing-subsequence.md — 최장 증가 부분수열
- [ ] subset-sum.md — 부분집합 합
- [ ] ordered-partition.md — 순서 있는 분할
- [ ] context-free-grammar-parsing.md — 문맥 자유 문법 파싱
- [ ] dp-tsp.md — DP로 접근하는 TSP

## Chapter 11 — NP-Completeness

- [ ] reduction.md — 환원
- [ ] decision-problem.md — 결정 문제
- [ ] p-vs-np.md — P vs NP
- [ ] np-complete.md — NP-완전
- [ ] np-hard.md — NP-난해
- [ ] satisfiability.md — 충족 가능성 문제
- [ ] 3-sat.md — 3-SAT
- [ ] vertex-cover.md — 정점 커버
- [ ] independent-set.md — 독립 집합
- [ ] hamiltonian-cycle.md — 해밀턴 순환
- [ ] clique.md — 클리크

## Chapter 12 — Dealing with Hard Problems

- [ ] approximation-algorithm.md — 근사 알고리즘
- [ ] vertex-cover-approximation.md — 정점 커버 근사
- [ ] euclidean-tsp.md — 유클리드 TSP
- [ ] set-cover.md — 집합 커버
- [ ] random-sampling.md — 랜덤 샘플링
- [ ] local-search.md — 지역 탐색
- [ ] simulated-annealing.md — 시뮬레이티드 어닐링
- [ ] genetic-algorithm.md — 유전 알고리즘
- [ ] quantum-computing-intro.md — 양자 컴퓨팅 개요

## Chapter 13 — How to Design Algorithms

- [ ] algorithm-design-process.md — 알고리즘 설계 방법론
- [ ] tech-interview-strategy.md — 테크 인터뷰 전략

---

진행 방식: 챕터 단위로 완료 후 다음 챕터 이동. 완료된 항목은 [x]로 표시.
