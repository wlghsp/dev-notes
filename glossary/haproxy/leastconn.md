# leastconn (최소 연결)

현재 활성 연결 수가 가장 적은 서버로 요청을 보내는 로드밸런싱 알고리즘.

## 동작

HAProxy가 각 서버의 현재 연결 수를 추적한다. 새 요청이 들어오면 연결이 가장 적은 서버를 선택한다.

연결 수가 같으면 roundrobin처럼 순서대로 선택한다.

## roundrobin과의 차이

roundrobin은 처리 중인 요청 수를 신경 쓰지 않고 순서대로 보낸다. 요청 처리 시간이 균일하면 문제없지만, 처리 시간 편차가 크면 특정 서버에 미완료 연결이 쌓인다.

leastconn은 이 문제를 해결한다. 처리가 느린 서버는 연결이 많이 쌓여 새 요청을 덜 받게 된다.

## 언제 쓰나

WebSocket, gRPC 스트리밍, DB 연결처럼 오래 유지되는 연결(long-lived connection)을 다룰 때 적합하다. HTTP 단건 요청처럼 연결이 빨리 끝나는 경우엔 roundrobin과 차이가 없다.

```
backend ws_back
    balance leastconn
    server ws1 192.168.1.10:8080 check
    server ws2 192.168.1.11:8080 check
```

참고: roundrobin.md, source.md
