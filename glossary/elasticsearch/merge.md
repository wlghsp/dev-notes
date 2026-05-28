# Merge (머지)

여러 개의 작은 Segment를 하나의 큰 Segment로 합치는 백그라운드 작업.

## 왜 필요한가

Refresh가 실행될 때마다 새 Segment가 생성된다. 시간이 지날수록 작은 Segment가 누적되고, 검색 요청마다 모든 Segment를 탐색해야 하므로 검색 성능이 떨어진다.

Merge는 이 문제를 해결한다. 작은 Segment를 합쳐 Segment 수를 줄이고, 그 과정에서 삭제 표시(tombstone)된 문서를 실제로 제거해 디스크 공간도 회수한다.

## 흐름

1. Elasticsearch가 작은 Segment들을 선택한다.
2. 선택된 Segment들을 합쳐 새로운 큰 Segment를 생성한다.
3. 삭제 표시된 문서는 새 Segment에 포함하지 않는다.
4. 새 Segment가 완성되면 기존 Segment들을 삭제한다.

## 자동 Merge

Merge는 Elasticsearch가 자동으로 판단해 백그라운드에서 실행한다. 개발자가 직접 트리거할 필요는 없다.

수동으로 강제 실행하는 것도 가능하다.

```
POST /products/_forcemerge?max_num_segments=1
```

`max_num_segments=1`은 인덱스의 모든 Segment를 하나로 합치는 가장 적극적인 최적화다. 읽기 전용 인덱스(더 이상 색인이 없는 인덱스)에 적용하면 검색 성능을 극대화할 수 있다. 실행 중에는 리소스를 많이 소모하므로 운영 중 트래픽이 적은 시간대에 한다.

## 삭제된 문서와 디스크

Elasticsearch에서 문서를 삭제해도 디스크 공간이 즉시 해제되지 않는다. Merge가 실행돼야 삭제 표시된 문서가 실제로 제거되고 디스크 공간이 회수된다.

참고: segment.md, refresh.md
