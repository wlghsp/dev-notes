# compaction
참고: lsm-tree.md, sstable.md, write-amplification.md

---

LSM-tree 기반 저장 엔진에서 여러 SSTable 세그먼트를 병합해 불필요한 데이터를 제거하고 디스크 공간을 회수하는 과정이다.

## 왜 필요한가

LSM-tree는 쓰기를 항상 새 파일에 append하기 때문에, 같은 키에 대한 갱신이나 삭제가 쌓이면 오래된 버전 데이터가 여러 파일에 흩어진다. compaction 없이 두면 디스크 공간이 끝없이 늘어나고, 읽기 시 확인해야 할 파일 수가 많아진다.

## 동작 방식

두 개 이상의 SSTable 파일을 읽어서 merge sort로 합친다. 같은 키가 여러 파일에 있으면 가장 최신 버전만 남기고 나머지는 버린다. 삭제된 키는 tombstone이 있으면 해당 키 자체를 최종 결과에서 제외한다. 병합 결과를 새 SSTable 파일로 쓰고, 원본 파일들을 삭제한다.

## 전략: 어떤 파일을 언제 병합하느냐

size-tiered compaction은 비슷한 크기의 파일들을 하나로 합친다. 공간 증폭은 크지만 write amplification이 낮다. Cassandra가 기본적으로 이 방식을 쓴다.

leveled compaction은 레벨 단위로 파일 수를 제한한다. 각 레벨의 총 크기가 임계치를 넘으면 다음 레벨로 내려보낸다. 공간 증폭이 작고 읽기 성능이 좋지만 write amplification이 높다. LevelDB, RocksDB의 기본 방식이다.

## compaction과 서비스 성능

compaction은 백그라운드에서 돌지만 디스크 I/O와 CPU를 쓴다. 쓰기 속도가 compaction 속도보다 빠르면 파일이 계속 쌓여서 읽기 성능이 나빠진다. 이를 write stall이라고 하며, 운영 환경에서 모니터링이 필요하다.
