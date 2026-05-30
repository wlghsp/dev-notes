# Node (노드)

Elasticsearch 클러스터를 구성하는 단일 서버 인스턴스. 각 노드는 역할에 따라 담당하는 일이 다르다.

## 노드 역할 종류

Master Node: 클러스터 전체 상태를 관리한다. 인덱스 생성/삭제, Shard 할당, 노드 추가/제거 등 클러스터 메타데이터를 책임진다. 실제 데이터를 처리하지 않는다.

데이터가 많아도 Master Node가 느려지면 클러스터 전체에 영향을 미친다. 그래서 프로덕션에서는 Master Node를 전용으로 분리한다.

Data Node: 실제 Shard를 저장하고 색인·검색을 처리한다. 데이터와 트래픽이 집중되는 노드다. 스케일아웃은 주로 Data Node를 늘리는 방식으로 한다.

Coordinating Node: 클라이언트 요청을 받아 관련 Shard들에 분산하고 결과를 모아 반환하는 역할만 한다. 데이터를 저장하지 않고 요청 라우팅과 집계만 담당한다. 모든 노드가 기본적으로 Coordinating 역할을 겸한다.

Ingest Node: 색인 전에 문서를 전처리(파이프라인)하는 역할. Logstash 없이 간단한 데이터 변환을 Elasticsearch 안에서 처리하고 싶을 때 사용.

## 역할 분리의 이유

작은 클러스터에서는 모든 노드가 모든 역할을 겸해도 된다. 클러스터가 커지면 Master/Data/Coordinating 역할을 분리해 각 노드가 한 가지 일에 집중하게 한다.

Master Node가 Data 역할을 겸하면, 대량 색인 중 GC 압박으로 Master가 응답 못하는 상황이 생길 수 있다.

## Master 선출

Master Node는 하나만 활성화된다(Active Master). 나머지 Master-eligible Node들은 대기 상태다. Active Master가 죽으면 나머지 중 하나가 선출된다.

Split-brain(두 노드가 동시에 Master라고 판단하는 상황)을 방지하기 위해 Master-eligible Node 수를 홀수로 유지한다. 최소 3개 권장.

참고: cluster.md, shard.md, shard-allocation.md
