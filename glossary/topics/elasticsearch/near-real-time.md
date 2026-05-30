# Near Real-Time (NRT, 준실시간)

색인된 문서가 검색에 반영되기까지 짧은 지연(기본 1초)이 있는 특성. 완전한 실시간은 아니지만 거의 실시간에 가깝다는 의미다.

## 왜 실시간이 아닌가

Elasticsearch는 성능을 위해 색인된 문서를 바로 디스크에 쓰지 않는다. in-memory buffer에 쌓아두다가 Refresh(기본 1초마다)가 실행될 때 새 Segment를 만들어 검색 가능한 상태로 전환한다.

이 1초 지연이 "Near" Real-Time인 이유다.

## 실제 의미

문서를 색인하고 나서 최대 1초 안에 검색 결과에 나타난다. 대부분의 애플리케이션에서 이 정도 지연은 문제가 없다.

실시간 검색이 필요한 경우(예: 방금 색인한 문서를 즉시 조회해야 하는 API 테스트)에는 수동으로 Refresh를 호출하거나, 색인 요청에 `refresh=true` 파라미터를 넣는다.

```
PUT /products/_doc/1?refresh=true
{
  "name": "노트북"
}
```

단, 매 요청마다 `refresh=true`를 쓰면 작은 Segment가 계속 생겨 성능 문제가 생긴다. 프로덕션에서는 피해야 한다.

참고: refresh.md, segment.md
