# modeling (문제 모델링)

실세계 응용 문제를 잘 정의된 추상 구조로 표현하는 과정. Skiena는 이를 "알고리즘 솔루션을 향한 가장 중요한 단계"로 부른다.

실세계 문제는 "위젯 최적화 문제" 같은 형태로 온다. 이를 알고리즘 문헌에서 다루는 추상 구조로 변환해야 기존에 알려진 알고리즘을 적용할 수 있다.

핵심 추상 구조들:
- Permutations(순열) — arrangement, tour, ordering, sequence를 찾는 문제
- Subsets(부분집합) — cluster, collection, committee, selection을 찾는 문제
- Trees(트리) — hierarchy, dominance, ancestor/descendant 관계를 다루는 문제
- Graphs(그래프) — network, circuit, web, relationship을 다루는 문제
- Points(점) — location, site, position, data record를 다루는 문제
- Polygons(다각형) — shape, region, configuration, boundary를 다루는 문제
- Strings(문자열) — text, character, pattern, label을 다루는 문제

이 구조들은 모두 재귀적(recursive)으로 분해된다. 순열에서 원소 하나를 빼면 더 작은 순열, 그래프에서 정점 하나를 빼면 더 작은 그래프가 된다.

모델링의 함정: 문제를 하나의 추상 구조로 환원할 때 세부사항이 맞지 않을 수 있다. 아무리 좋은 알고리즘이라도 문제를 잘못 모델링하면 틀린 답이 나온다. War Story에서 복권 티켓 문제를 set cover로 모델링했다가 "covering" 기준을 잘못 정의해 처음에는 틀린 답이 나왔다.

참고: combinatorial-object.md, algorithm.md
