# Sticky Session (세션 고정)

같은 클라이언트가 항상 같은 백엔드 서버로 연결되도록 고정하는 기능. Session Persistence라고도 한다.

## 왜 필요한가

세션 데이터를 서버 메모리에 저장하는 구조(세션 기반 인증 등)에서는 같은 사용자가 항상 같은 서버로 가야 한다. 다른 서버로 가면 세션을 찾지 못해 로그인이 풀린다.

## HAProxy에서의 구현 — 쿠키 기반

HAProxy가 응답에 쿠키를 심어 서버를 식별한다.

```
backend api_back
    balance roundrobin
    cookie SERVERID insert indirect nocache
    server api1 192.168.1.10:8080 check cookie api1
    server api2 192.168.1.11:8080 check cookie api2
```

- `cookie SERVERID insert`: `SERVERID`라는 쿠키를 응답에 추가
- `indirect`: 클라이언트에게 보내는 응답에만 쿠키를 추가하고, 서버에는 전달하지 않음
- `nocache`: 중간 캐시에서 이 쿠키를 캐싱하지 못하게 함
- `cookie api1`: 이 서버를 식별하는 쿠키 값

첫 요청은 roundrobin으로 배분된다. 이후 요청부터는 `SERVERID=api1` 쿠키를 보고 api1으로 고정된다.

## source 기반과의 차이

source 알고리즘은 IP 해싱으로 서버를 고정한다. TCP 모드에서도 쓸 수 있지만 NAT 뒤에 사용자가 몰리면 특정 서버에 집중될 수 있다.

쿠키 기반은 HTTP 모드에서만 쓸 수 있지만 사용자 단위로 정밀하게 고정된다.

## 한계

Sticky Session은 서버 간 부하 분산 효율을 낮춘다. 특정 서버에 활성 사용자가 집중되면 그 서버만 바쁘다. 근본적으로는 세션을 Redis 같은 외부 저장소에 두어 어떤 서버로 가도 세션을 찾을 수 있게 하는 것이 더 나은 설계다.

참고: source.md, roundrobin.md
