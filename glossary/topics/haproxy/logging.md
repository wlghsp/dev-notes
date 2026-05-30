# Logging (로깅)

HAProxy가 요청과 연결에 대한 정보를 기록하는 기능. 기본적으로 syslog로 전송한다.

## 기본 설정

```
global
    log /dev/log local0 info
    log /dev/log local1 notice
```

`local0`, `local1`은 syslog facility다. `/dev/log`는 로컬 syslog 소켓 경로다.

## 로그 포맷

HTTP 모드 기본 로그 한 줄 예시:

```
192.168.1.5:45231 [28/May/2026:10:30:01.234] http_front api_back/api1 0/0/1/52/53 200 1234 - - ---- 3/3/2/1/0 0/0 "GET /api/users HTTP/1.1"
```

순서대로:
- 클라이언트 IP:포트
- 타임스탬프
- frontend 이름
- backend/server 이름
- 타이밍(Tq/Tw/Tc/Tr/Tt ms) — 대기/큐/연결/응답/전체 소요 시간
- HTTP 상태 코드
- 응답 바이트 수
- 세션 상태 플래그
- 연결 수 현황
- 요청 라인

## 커스텀 로그 포맷

```
frontend http_front
    log-format "%ci:%cp [%t] %ft %b/%s %Tq/%Tw/%Tc/%Tr/%Tt %ST %B %tsc %ac/%fc/%bc/%sc/%rc %{+Q}r"
```

특정 헤더를 로그에 포함할 수도 있다.

```
http-request capture req.hdr(User-Agent) len 100
http-request capture req.hdr(X-Request-ID) len 36
```

`capture`로 지정한 헤더 값이 로그의 `{cap}` 자리에 기록된다.

## 타이밍 필드 활용

Tq, Tw, Tc, Tr, Tt 타이밍 필드는 성능 진단에 핵심이다.

- Tw(queue 대기 시간)가 길면 서버 maxconn이 부족한 것
- Tc(연결 시간)가 길면 네트워크 또는 서버 accept 지연
- Tr(서버 응답 시간)가 길면 백엔드 처리가 느린 것

참고: maxconn.md, queue.md
