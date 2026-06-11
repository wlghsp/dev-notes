# lsm-tree
참고: sstable.md, compaction.md, write-amplification.md, b-tree.md

---

Log-Structured Merge Tree. 쓰기를 항상 순차적으로 처리해서 write throughput을 극대화하는 저장 구조다. LevelDB, RocksDB, Cassandra, HBase가 이 구조를 기반으로 한다.

## 구조

세 단계로 나뉜다.

1. memtable — 인메모리 정렬 자료구조(레드-블랙 트리 또는 스킵리스트). 새 쓰기가 오면 여기에 먼저 쌓인다.
2. immutable memtable — memtable이 임계 크기를 넘으면 읽기 전용으로 전환하고, 새 memtable을 생성한다.
3. SSTable — immutable memtable을 디스크에 정렬된 상태로 내려쓴 파일. L0 → L1 → L2 순으로 레벨이 깊어지며, 레벨이 깊을수록 더 오래되고 더 큰 파일들이 모인다.

쓰기 손실을 막기 위해 WAL(Write-Ahead Log)에 먼저 기록한다. 장애 시 WAL을 재생해서 memtable을 복구한다.

## 읽기: 여러 단계를 확인해야 한다

읽기 요청이 오면 memtable → L0 SSTable → L1 → ... 순으로 찾는다. 가장 최근 쓰기가 이기기 때문에 찾자마자 반환한다. 존재하지 않는 키는 모든 레벨을 다 확인해야 알 수 있어서 비효율적이다. 이 문제를 블룸 필터(bloom filter)로 완화한다. 블룸 필터는 "이 키가 이 SSTable에 확실히 없다"는 것을 빠르게 알려줘서 불필요한 디스크 읽기를 건너뛸 수 있게 한다.

## B-tree와의 비교

LSM-tree는 쓰기에 유리하고 B-tree는 읽기에 유리하다. LSM-tree는 항상 순차 쓰기라 HDD에서도 빠르고 SSD 수명도 아낀다. 반면 읽기는 여러 레벨을 확인해야 해서 B-tree보다 느릴 수 있다. compaction 작업이 백그라운드에서 동작하므로 실시간 쓰기 성능에 간헐적으로 영향을 줄 수 있다는 점도 트레이드오프다.
