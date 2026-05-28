# Rate Limiting (요청 제한)

특정 클라이언트나 조건의 요청 수를 시간 단위로 제한하는 기능. HAProxy는 Stick Table을 이용해 rate limiting을 구현한다.

## Stick Table

HAProxy의 인메모리 상태 저장소. IP별 요청 수, 연결 수, 세션 수 등을 추적하는 데 쓰인다.

```
backend rate_limit_table
    stick-table type ip size 1m expire 10s store http_req_rate(10s)
```

- `type ip`: IP 주소를 키로 사용
- `size 1m`: 최대 100만 개 항목
- `expire 10s`: 10초 동안 활동 없으면 삭제
- `store http_req_rate(10s)`: 10초 동안의 요청 수를 추적

## 구현 예시

```
frontend http_front
    bind *:80

    # Stick Table 업데이트
    stick-table type ip size 1m expire 10s store http_req_rate(10s)
    http-request track-sc0 src

    # 10초에 100번 초과 시 차단
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 100 }

    default_backend api_back
```

`sc_http_req_rate(0)`는 Stick Table 0번에 기록된 해당 IP의 요청 수다.

## 연결 수 제한

요청 수가 아니라 동시 연결 수를 제한할 수도 있다.

```
stick-table type ip size 1m expire 30s store conn_cur
http-request track-sc0 src
http-request deny if { sc_conn_cur(0) gt 10 }
```

## 한계

HAProxy가 여러 프로세스로 실행되거나 여러 노드로 이중화된 경우, Stick Table은 각 프로세스/노드가 독립적으로 관리한다. 정확한 글로벌 rate limiting이 필요하면 외부 솔루션(Redis + 애플리케이션 레이어)과 조합이 필요하다.

참고: connection-limiting.md
