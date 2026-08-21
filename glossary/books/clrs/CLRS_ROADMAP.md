# CLRS (Introduction to Algorithms, 3rd Edition) 로드맵

Cormen, Leiserson, Rivest, Stein 저. glossary/topics/algorithm/에서 다루던 알고리즘 정당성 증명 세트(귀류법, 교환 논증 등)를 이 책으로 체계적으로 이어간다.

## 이미 topics/algorithm/에 있는 개념과 책 속 위치

- proof-by-contradiction, induction, extremal-principle, pigeonhole-principle → Appendix A/B/C (수학적 배경), 대부분 본문 증명 안에 도구로 녹아 있고 별도 장은 아님
- loop-invariant → 2장 Getting Started (2.1 Insertion sort에서 최초 도입, 이후 책 전체에서 반복 사용)
- exchange-argument, optimal-substructure → 15장 Dynamic Programming, 16장 Greedy Algorithms
- monotonicity-argument → 이 책에는 투 포인터를 별도로 다루는 장이 없음. 9장(Medians and Order Statistics)의 selection 알고리즘 등에서 비슷한 논증이 부분적으로 등장

## 학습 순서 (Part 단위)

### Part I. Foundations — 기초 다지기
- 1장 The Role of Algorithms in Computing
- 2장 Getting Started (Insertion sort, 알고리즘 분석 시작 — loop-invariant 재확인 지점)
- 3장 Growth of Functions (점근 표기법)
- 4장 Divide-and-Conquer — **재귀 관계/마스터 정리, 다음 단계로 짚었던 부분**
- 5장 Probabilistic Analysis and Randomized Algorithms — 확률적 논증 축

### Part II. Sorting and Order Statistics
- 6장 Heapsort, 7장 Quicksort, 8장 Sorting in Linear Time, 9장 Medians and Order Statistics

### Part III. Data Structures
- 10~14장 (기본 자료구조, 해시 테이블, BST, 레드-블랙 트리, 자료구조 확장)

### Part IV. Advanced Design and Analysis Techniques
- 15장 Dynamic Programming — exchange-argument, optimal-substructure와 직결
- 16장 Greedy Algorithms — 교환 논증이 이 책에서 어떻게 정식화되는지 확인
- 17장 Amortized Analysis — 로드맵에 남겨뒀던 상환 분석

### Part V. Advanced Data Structures
- 18~21장 (B-트리, 피보나치 힙, van Emde Boas 트리, 분리 집합)

### Part VI. Graph Algorithms — **그래프 기본 정리, 다음 단계로 짚었던 부분**
- 22장 Elementary Graph Algorithms (BFS, DFS, 위상 정렬)
- 23장 Minimum Spanning Trees (Kruskal, Prim — cut property)
- 24장 Single-Source Shortest Paths (Bellman-Ford, Dijkstra — relaxation 정당성)
- 25장 All-Pairs Shortest Paths
- 26장 Maximum Flow

### Part VII. Selected Topics
- 27~33장 (멀티스레드, 행렬 연산, 선형 계획법, FFT, 정수론, 문자열 매칭, 계산 기하)
- 34장 NP-Completeness — **문제 자체의 한계를 다루는 축, 다음 단계로 짚었던 부분**
- 35장 Approximation Algorithms

## 보조 자료

- Algorithms Unlocked (Cormen, 2013) — CLRS의 저자가 쓴 대중서. 수학적 증명보다 알고리즘이 무엇을 하는지 설명하는 쪽에 가까워서, CLRS 내용이 어려울 때 감을 잡는 용도로만 참고. 별도 로드맵/챕터 파일은 만들지 않는다

## 진행 상황

- 1~3장은 건너뛰고 4장(Divide-and-Conquer)부터 시작. topics/algorithm/에서 loop-invariant, induction 등 관련 개념을 이미 다뤘고 점근 표기법도 익숙해서 반복 학습을 피함. 필요해지면 나중에 되짚는다
- 문서 구조는 CLAUDE.md의 "기술 서적 챕터별 문서 구조" 규칙을 따른다 (키워드 파일 + 챕터 종합 문서)
- 챕터 완료 후 실제 생성된 파일만 이 로드맵에 추가할 것 (예상 키워드는 미리 적지 않음)

## 4장 Divide-and-Conquer — 완료

키워드 파일: recurrence-relation.md, substitution-method.md, recursion-tree-method.md, master-theorem.md
챕터 종합 문서: ch04-divide-and-conquer.md (maximum-subarray, Strassen 예시로 정리)
4.6절(마스터 정리 증명)까지 포함해서 완료. 세 가지 풀이법(치환법/재귀 트리/마스터 정리)의 관계와 마스터 정리 증명 개요는 master-theorem.md에 통합
불균등 분할 재귀식을 위한 Akra-Bazzi 방법은 챕터 노트에서 이름만 확인, 깊게는 안 다룸 — 필요해지면 별도 학습
