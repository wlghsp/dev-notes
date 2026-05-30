# check (헬스체크)

`server` 지시어에 붙이는 키워드. 이 옵션이 있으면 HAProxy가 해당 서버에 주기적으로 상태 확인 요청을 보내고, 실패하면 트래픽에서 자동으로 제외한다.

```
backend api_back
    server api1 192.168.1.10:8080 check
    server api2 192.168.1.11:8080 check
```

`check`가 없으면 서버가 죽어도 HAProxy는 모르고 계속 트래픽을 보낸다.

## 세부 옵션

```
server api1 192.168.1.10:8080 check inter 2s rise 2 fall 3
```

- `inter 2s` — 2초마다 체크
- `rise 2` — 2번 연속 성공하면 서버 UP으로 판단
- `fall 3` — 3번 연속 실패하면 서버 DOWN으로 판단, 트래픽 제외

## 체크 방식

기본은 TCP 연결 시도다. 포트가 열려있으면 정상으로 판단한다.

HTTP 모드에서는 특정 경로로 HTTP 요청을 보내 응답 코드를 확인할 수 있다.

```
backend api_back
    option httpchk GET /health
    server api1 192.168.1.10:8080 check
```

`/health`가 2xx를 반환하면 정상, 그 외엔 실패로 판단한다.

참고: balance.md
