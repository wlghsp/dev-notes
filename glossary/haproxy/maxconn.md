# maxconn (최대 연결 수)

HAProxy가 동시에 처리할 수 있는 최대 연결 수 제한. 적용 레벨에 따라 의미가 다르다.

## 레벨별 의미

global maxconn: HAProxy 프로세스 전체의 최대 연결 수. 이 수를 초과하면 OS 레벨에서 새 연결을 거부한다.

```
global
    maxconn 50000
```

frontend maxconn: 특정 frontend에서 받을 수 있는 최대 연결 수. 초과 연결은 OS의 accept queue에서 대기한다.

```
frontend http_front
    maxconn 10000
```

backend server maxconn: 특정 서버로 보낼 수 있는 최대 동시 연결 수. 초과 요청은 HAProxy의 queue에 들어간다.

```
backend api_back
    server api1 192.168.1.10:8080 check maxconn 100
```

## server maxconn과 queue

서버의 maxconn이 100이고 현재 100개 연결이 모두 사용 중이면, 새 요청은 서버로 가지 않고 HAProxy 내부 queue에 쌓인다. 기존 연결이 끝나면 queue에서 꺼내 서버로 보낸다.

queue에서 기다리는 시간은 `timeout queue`로 제한한다. 제한 시간 내에 서버를 배정받지 못하면 503을 반환한다.

서버가 감당할 수 있는 수준 이상의 연결이 몰리는 것을 막는 backpressure 역할을 한다.

## 왜 server maxconn이 필요한가

백엔드 서버(예: Tomcat)가 동시에 처리할 수 있는 스레드 수는 제한돼 있다. 그 수를 넘으면 서버 내부 queue가 쌓이고 응답이 느려진다. HAProxy의 server maxconn을 서버 처리 용량에 맞게 설정하면 서버 과부하를 예방할 수 있다.

참고: timeout.md, queue.md
