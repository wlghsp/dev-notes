# graph-model
참고: relational-model.md, document-model.md

---

데이터를 vertex(노드)와 edge(간선)로 표현하는 모델이다. many-to-many 관계가 매우 복잡하게 얽혀 있을 때 가장 자연스러운 선택이다.

## 관계형 모델과의 차이

관계형 모델도 many-to-many를 표현할 수 있지만, 관계가 복잡해질수록 쿼리가 어색해진다. 그래프 모델은 어떤 vertex도 다른 어떤 vertex와 edge로 연결될 수 있어서, 관계 유형에 제한이 없다. Facebook처럼 사람, 장소, 이벤트, 댓글이 모두 하나의 그래프 안에서 다양한 edge로 연결되는 구조가 그 예다.

## property graph 모델

각 vertex는 고유한 ID, incoming/outgoing edge 목록, 속성(key-value 쌍)을 가진다. 각 edge는 고유한 ID, tail vertex, head vertex, 관계를 설명하는 label, 속성을 가진다. 이 구조를 관계형 DB로 표현하면 vertices 테이블과 edges 테이블 두 개로 나타낼 수 있다.

## 쿼리 언어

그래프를 위한 선언형 쿼리 언어들이 있다. Cypher는 Neo4j에서 쓰는 언어로, 화살표 표기법으로 패턴을 선언한다. SPARQL은 triple-store(subject-predicate-object 형태로 저장하는 모델)를 위한 쿼리 언어다. SQL로도 그래프 쿼리를 표현할 수 있지만, `WITH RECURSIVE` 같은 문법이 필요하고 Cypher에 비해 훨씬 장황하다.

## 언제 쓰는가

데이터 간 연결 자체가 핵심인 경우다. 소셜 그래프, 도로/철도 네트워크, 지식 그래프 등이 대표적이다. 데이터가 주로 one-to-many 트리 구조면 document model이, 단순한 many-to-many면 relational model이 더 적합하다.
