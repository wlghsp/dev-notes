# algorithm glossary 로드맵

알고리즘 문제를 풀 때 "왜 이 풀이가 맞는가"를 설명하는 증명 기법과 사고 도구를 정리하는 용어집.

## 완료

- proof-by-contradiction.md — 귀류법
- exchange-argument.md — 교환 논증 (그리디 최적성 증명)
- induction.md — 수학적 귀납법
- loop-invariant.md — 반복문 불변량
- monotonicity-argument.md — 단조성 논증 (투 포인터/슬라이딩 윈도우)
- extremal-principle.md — 극단적 원리 (최소/최대 반례)
- pigeonhole-principle.md — 비둘기집 원리
- optimal-substructure.md — 최적 부분 구조 (DP 전제 조건)

## 다음 후보

- overlapping-subproblems.md — 겹치는 부분 문제. optimal-substructure와 함께 DP를 쓸 수 있는 두 번째 전제 조건. 이게 없으면 그냥 분할 정복으로 충분함
- amortized-analysis.md — 상환 분석. 반복문 하나가 가끔 비싸도 전체로 보면 평균이 싸다는 걸 증명하는 방법 (예: 동적 배열의 resize, union-find의 경로 압축)
- correctness-vs-complexity.md — 정확성 증명과 시간복잡도 분석이 별개라는 걸 명확히 하는 짧은 정리. 증명 기법 자체는 아니지만 지금 세트를 왜 배우는지 다시 짚어주는 역할
- graph-invariant-examples.md — loop-invariant를 그래프 탐색(BFS/DFS)에 적용한 예시 모음. 개념은 이미 있는 loop-invariant.md에 있으니, 이건 새 개념이라기보다 예시 확장 여부를 나중에 판단

## 참고 자료

- 지금까지 학습에는 별도 외부 자료 없이 이미 알고 있는 알고리즘 지식과 실제 푼 문제(숫자 게임 등)를 예시로 씀
- 더 체계적으로 보고 싶다면 CLRS(Introduction to Algorithms) 책의 각 장 도입부에 있는 정당성 증명 스타일이 이 세트와 잘 맞음. 필요해지면 book_study_order 흐름에 편입할지 그때 판단
