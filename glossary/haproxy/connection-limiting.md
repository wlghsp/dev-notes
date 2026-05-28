# Connection Limiting (연결 수 제한)

특정 IP나 조건의 동시 연결 수를 제한하는 기능. Rate Limiting이 시간 단위 요청 수를 제한한다면, Connection Limiting은 현재 맺어진 연결 수를 기준으로 제한한다.

## Rate Limiting과의 차이

Rate Limiting: "10초에 100번 이상 요청하면 차단" — 시간 기반.
Connection Limiting: "동시에 10개 이상 연결하면 차단" — 현재 상태 기반.

DDoS 패턴에 따라 다르게 적용한다. 연결을 맺고 오래 유지하는 공격엔 Connection Limiting이 효과적이고, 빠르게 요청을 반복하는 공격엔 Rate Limiting이 맞다.

## 구현 예시

```
frontend http_front
    bind *:80
    stick-table type ip size 1m expire 30s store conn_cur,conn_rate(10s)
    tcp-request connection track-sc0 src
    tcp-request connection reject if { sc_conn_cur(0) gt 20 }

    default_backend api_back
```

`conn_cur`: 현재 해당 IP의 동시 연결 수.
`tc-request connection`: HTTP 파싱 전 TCP 레벨에서 판단. 더 이른 단계에서 차단한다.

## frontend maxconn과의 차이

maxconn은 HAProxy 전체 또는 서버 하나의 최대 연결 수 상한선이다. 초과하면 queue에서 대기한다.

Connection Limiting은 특정 IP나 조건에 대한 제한이다. 특정 IP가 연결을 독점하는 것을 막는다.

참고: rate-limiting.md, maxconn.md
