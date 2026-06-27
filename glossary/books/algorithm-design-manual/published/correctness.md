# correctness (올바름)

알고리즘이 모든 가능한 입력 인스턴스에 대해 요구된 출력을 생성함을 의미한다.

올바름은 "잘 동작하는 것 같다"는 느낌으로는 증명되지 않는다. 합리적으로 보이는 알고리즘이 틀릴 수 있다는 게 이 책의 핵심 메시지 중 하나다. TSP의 nearest-neighbor 휴리스틱이나 movie scheduling의 shortest-job-first 휴리스틱이 모두 반례가 존재한다.

올바름을 증명하는 주요 방법:
1. 수학적 귀납법(induction) — 재귀적/증분적 알고리즘에 특히 유효
2. 모순에 의한 증명(proof by contradiction)
3. 직접 증명

올바름을 반증하는 방법:
- 반례(counterexample) 찾기 — 한 개의 반례로 충분하다

올바름 증명 시 흔한 실수:
- 경계 오류(boundary error) — 기저 조건이나 극단 입력을 제대로 처리 안 함
- 확장 오류 — 최적해에 원소를 추가하면 이전의 최적해를 포함하지 않을 수 있음

참고: counterexample.md, induction.md
