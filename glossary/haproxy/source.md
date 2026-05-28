# source (소스 IP 해싱)

클라이언트의 소스 IP 주소를 해싱해 항상 같은 서버로 보내는 로드밸런싱 알고리즘.

## 동작

같은 IP에서 오는 요청은 서버가 살아있는 한 항상 같은 서버로 라우팅된다. 서버 수가 바뀌면 해시 결과가 달라질 수 있다.

```
backend api_back
    balance source
    server api1 192.168.1.10:8080 check
    server api2 192.168.1.11:8080 check
```

## 세션 고정과의 관계

sticky session(쿠키 기반)과 목적은 같다 — 같은 클라이언트를 같은 서버로 보내는 것. 하지만 방식이 다르다.

source는 IP 기반이라 쿠키를 심을 수 없는 TCP 모드에서도 쓸 수 있다. 반면 NAT 뒤에 클라이언트가 몰려있으면 같은 IP에서 오는 요청이 전부 하나의 서버에 집중된다.

sticky session은 HTTP 모드에서만 쓸 수 있지만 더 정밀하게 동작한다.

## 언제 쓰나

TCP 모드(MySQL, Redis 앞단)에서 클라이언트마다 연결을 고정해야 할 때. HTTP 모드라면 쿠키 기반 sticky session을 쓰는 게 더 안정적이다.

참고: roundrobin.md, leastconn.md, sticky-session.md
