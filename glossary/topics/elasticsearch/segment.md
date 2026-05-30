# Segment (세그먼트)

Lucene이 데이터를 디스크에 저장하는 물리적 단위. 하나의 Shard는 여러 Segment로 구성된다.

## 불변성(Immutable)

Segment는 한번 생성되면 수정되지 않는다. 새 문서가 들어오면 새 Segment가 만들어지고, 기존 문서를 "수정"하면 기존 Segment에 삭제 표시(tombstone)를 하고 새 Segment에 새 버전을 쓴다.

이 불변성 덕분에 동시성 제어가 단순해지고, OS 파일 캐시를 효율적으로 활용할 수 있다.

## Segment가 쌓이는 문제

문서가 계속 들어올수록 작은 Segment가 쌓인다. Segment가 많아지면 검색 시 모든 Segment를 뒤져야 하므로 성능이 떨어진다.

이를 해결하기 위해 Merge가 작은 Segment 여러 개를 하나의 큰 Segment로 합친다. Merge 과정에서 삭제 표시된 문서도 실제로 제거된다.

참고: merge.md

## 검색 흐름

검색 요청이 들어오면 Shard 안의 모든 Segment에서 각각 검색하고 결과를 합친다. 그래서 Segment 수가 많을수록 검색 비용이 증가한다.

## 메모리 구조

새로 색인된 문서는 먼저 메모리의 in-memory buffer에 쌓인다. Refresh가 실행되면 buffer를 비우고 새 Segment를 만들어 검색 가능 상태로 만든다. Flush가 실행되면 Segment가 디스크에 영구 저장된다.

참고: refresh.md, flush.md, merge.md, inverted-index.md, shard.md
