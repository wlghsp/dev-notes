# Retry / Redispatch (재시도 / 재분배)

백엔드 서버 연결이 실패했을 때 HAProxy가 자동으로 재시도하거나 다른 서버로 요청을 넘기는 동작.

## Retry

서버 연결이 실패하거나 연결 중 오류가 발생하면 같은 서버에 재시도한다.

```
backend api_back
    retries 3
    server api1 192.168.1.10:8080 check
    server api2 192.168.1.11:8080 check
```

`retries 3`: 같은 서버에 최대 3번 재시도한다. 3번 모두 실패하면 그 서버를 down으로 표시하고 오류를 반환한다.

재시도는 연결 수립 단계의 실패에만 적용된다. 서버가 연결은 받았는데 요청 처리 중 오류를 반환한 경우엔 기본적으로 재시도하지 않는다.

## Redispatch

재시도 후에도 실패하면 다른 서버로 요청을 넘긴다.

```
backend api_back
    retries 3
    option redispatch
    server api1 192.168.1.10:8080 check
    server api2 192.168.1.11:8080 check
```

`option redispatch`: 마지막 재시도에서 다른 서버를 선택한다. Sticky Session을 사용 중이더라도 실패한 서버 대신 다른 서버로 보낸다.

## Retry vs Redispatch 흐름

```
요청 → api1 (실패) → retry → api1 (실패) → retry → api1 (실패)
    → redispatch → api2 (성공)
```

## 주의점

Retry는 idempotent한 요청(GET, HEAD)에는 안전하다. POST, PUT 같은 요청은 서버가 실제로 처리하고 응답만 못 보낸 경우일 수 있어, 재시도하면 중복 처리가 될 수 있다.

`option http-no-delay`나 retry 조건을 세밀하게 제어하려면 `retry-on` 지시어로 어떤 오류에서 재시도할지 지정할 수 있다.

참고: roundrobin.md, leastconn.md
