# Translog (트랜스로그)

Elasticsearch의 Write-Ahead Log. 색인된 데이터가 디스크에 완전히 저장되기 전에 장애가 나도 복구할 수 있도록 모든 변경 작업을 순서대로 기록하는 로그.

WAL의 역할과 동일하다. 참고: wal.md (database glossary)

## 왜 필요한가

새로 색인된 문서는 바로 디스크에 쓰이지 않는다. 성능을 위해 메모리 버퍼에 쌓여 있다가 Refresh → Flush 순서로 처리된다. Flush 전에 노드가 죽으면 버퍼의 데이터는 사라진다.

Translog는 이 구간의 안전망이다. 색인 요청이 들어올 때마다 Translog에 먼저 기록하기 때문에, 장애 후 재시작 시 Translog를 재실행해 데이터를 복구한다.

## 흐름

1. 색인 요청 도착
2. Translog에 기록 (디스크, fsync)
3. in-memory buffer에 추가
4. Refresh → 새 Segment 생성 (검색 가능)
5. Flush → Segment 디스크 저장 완료
6. Flush 완료 시점의 Translog는 더 이상 필요 없어 잘라낸다(truncate)

## Translog와 내구성

기본적으로 매 요청마다 Translog를 fsync한다(`index.translog.durability: request`). 이 설정이면 노드 장애 시 데이터 유실이 없다.

성능을 위해 fsync를 비동기로 바꿀 수 있지만(`async`), 장애 시 마지막 fsync 이후의 데이터는 유실될 수 있다.

참고: segment.md, refresh.md, flush.md
