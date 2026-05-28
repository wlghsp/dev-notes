# Refresh (리프레시)

in-memory buffer에 쌓인 문서를 새 Segment로 만들어 검색 가능한 상태로 전환하는 작업.

색인된 문서가 검색에 잡히려면 Refresh가 실행돼야 한다.

## 흐름

1. 문서가 색인되면 in-memory buffer에 쌓인다.
2. Refresh 실행 → buffer를 비우고 새 Segment 생성.
3. 새 Segment가 검색 대상에 포함된다.

Refresh는 Segment를 파일 시스템 캐시(OS cache)에 쓰는 것이다. 아직 디스크에 fsync하지 않은 상태 — 그건 Flush의 역할이다.

## 기본 주기

기본값은 1초(`index.refresh_interval: 1s`). 색인 후 최대 1초 뒤에 검색에 잡힌다. 이것이 Elasticsearch가 "Near Real-Time" 검색이라 불리는 이유다.

참고: near-real-time.md

## 수동 Refresh

```
POST /products/_refresh
```

색인 직후 바로 검색 결과를 확인해야 하는 테스트 상황에서 수동으로 호출한다.

## 성능 트레이드오프

Refresh가 자주 실행될수록 작은 Segment가 많이 생긴다. Segment가 많으면 검색 성능이 떨어진다. 대량 색인 시에는 `refresh_interval: -1`로 비활성화하고 완료 후 활성화하는 패턴을 쓴다.

참고: segment.md, flush.md, near-real-time.md
