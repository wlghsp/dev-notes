# Tombstone

삭제됐다는 사실을 기록으로 남기는 마커. 실제 데이터를 즉시 지우는 대신, "이 항목은 삭제됐다"는 흔적을 남겨둔다.

묘비(tombstone)라는 이름 그대로 — 사람은 없지만 묘비는 남아 있는 상태.

## 왜 즉시 삭제하지 않는가

즉시 삭제가 불가능하거나 비싼 상황이 있다.

- 힙 내부 임의 위치 삭제는 O(n) 탐색이 필요하다 → 삭제 표시만 하고 pop 시점에 처리
- LSM-tree는 디스크에 이미 쓴 데이터를 제자리에서 수정할 수 없다 → tombstone 레코드를 새로 append
- 분산 시스템에서 "삭제됐다"는 사실 자체를 다른 노드에 전파해야 한다 → 흔적이 없으면 전파할 수 없음

공통 원인은 **append-only 구조**이거나, **삭제 사실을 전파해야 하는 구조**다.

## 맥락별 형태

**힙 / 우선순위 큐**
삭제 요청이 들어온 원소를 invalid로 표시해두고, pop할 때 건너뛴다.
참고: lazy-deletion.md

**LSM-tree (RocksDB, Cassandra 등)**
삭제 요청이 들어오면 해당 키에 tombstone 레코드를 새로 append한다.
이후 compaction 단계에서 tombstone과 원본 데이터를 함께 제거한다.
tombstone이 compaction 전까지 살아있는 동안에는 읽기 시 "삭제된 키"로 처리된다.

**Kafka log compaction**
특정 키에 대해 value가 null인 메시지를 보내면 tombstone이 된다.
컨슈머는 이 메시지를 보고 해당 키가 삭제됐다고 인식한다.
log compaction이 실행되면 해당 키의 이전 메시지들과 tombstone 자체도 제거된다.

## tombstone이 쌓이면

tombstone은 영구적이지 않다. 정리되지 않으면 문제가 생긴다.

- 힙: invalid 셋이 메모리를 점유, pop마다 건너뛰는 비용 누적
- LSM-tree: compaction 전까지 읽기 성능 저하, 스토리지 낭비
- Kafka: retention 기간 동안 tombstone도 디스크를 차지

그래서 각 시스템은 tombstone을 주기적으로 정리하는 메커니즘을 가지고 있다.
힙은 pop 시점, LSM-tree는 compaction, Kafka는 log compaction이 그 역할을 한다.
