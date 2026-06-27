# combinatorial-object (조합론적 객체)

알고리즘 문제를 기술하는 데 사용되는 핵심 추상 구조들. 실세계 문제를 이 구조들 중 하나로 표현하면 기존에 알려진 알고리즘을 적용할 수 있다.

7가지 기본 객체:
1. Permutations — 원소들의 순서를 나타낸다. arrangement, tour, ordering, sequence를 찾는 문제에서 등장
2. Subsets — 원소들의 선택을 나타낸다. cluster, collection, committee, selection을 찾는 문제에서 등장
3. Trees — 계층 관계를 나타낸다. hierarchy, dominance, taxonomy를 다루는 문제에서 등장
4. Graphs — 임의의 쌍 관계를 나타낸다. network, circuit, web, relationship을 다루는 문제에서 등장
5. Points — 기하 공간의 위치를 나타낸다. location, position, site, data record를 다루는 문제에서 등장
6. Polygons — 기하 공간의 영역을 나타낸다. shape, region, configuration, boundary를 다루는 문제에서 등장
7. Strings — 문자 시퀀스를 나타낸다. text, pattern, label을 다루는 문제에서 등장

이 객체들은 모두 재귀적으로 정의된다. 큰 객체에서 원소 하나를 제거하면 같은 유형의 더 작은 객체가 된다. 이 성질이 분할 정복이나 동적 프로그래밍 같은 설계 기법과 연결된다.

참고: modeling.md, recursive-object.md
