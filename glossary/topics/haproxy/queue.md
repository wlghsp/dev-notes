# Queue (대기열)

백엔드 서버의 maxconn이 꽉 찼을 때 요청을 즉시 거부하지 않고 잠시 대기시키는 HAProxy 내부 버퍼.

## 동작

서버의 maxconn이 100이고 현재 100개가 모두 처리 중인 상태에서 새 요청이 들어오면:

1. 서버로 바로 보내지 않고 HAProxy queue에 넣는다.
2. 기존 연결 중 하나가 끝나면 queue에서 꺼내 서버로 보낸다.
3. `timeout queue` 시간 내에 서버를 배정받지 못하면 503을 반환한다.

## 어느 수준에서 대기하나

server maxconn 기준 queue: 특정 서버 하나가 꽉 찼을 때. HAProxy가 같은 backend의 다른 서버에 여유가 있으면 그쪽으로 보낸다. 모든 서버가 꽉 찼을 때만 queue에 쌓인다.

## queue vs 서버 자체 queue

WAS(예: Tomcat)도 내부 accept queue를 가진다. HAProxy의 queue와 다른 점은 HAProxy queue는 HAProxy가 완전히 제어하기 때문에 timeout, 모니터링이 가능하다. 서버 자체 queue는 가시성이 없다.

server maxconn을 적절히 설정하면 트래픽이 서버 자체 queue가 아닌 HAProxy queue에서 대기하게 되고, 이를 통해 통계 페이지에서 대기 상황을 모니터링할 수 있다.

## 설정 예시

```
defaults
    timeout queue 10s    # queue에서 최대 10초 대기

backend api_back
    server api1 192.168.1.10:8080 check maxconn 100
    server api2 192.168.1.11:8080 check maxconn 100
```

참고: maxconn.md, timeout.md
