# Timeout (타임아웃)

HAProxy에서 연결의 각 단계마다 적용되는 시간 제한. 종류가 여러 개고 각각 다른 구간을 담당한다.

## 주요 타임아웃 종류

`timeout connect`: HAProxy가 백엔드 서버에 TCP 연결을 맺을 때까지 기다리는 시간. 보통 짧게 설정한다 (3s~5s). 이 시간이 넘으면 서버가 죽은 것으로 판단하고 다른 서버를 시도한다.

`timeout client`: 클라이언트 쪽 비활성 시간 제한. 클라이언트가 데이터를 보내거나 받지 않고 멈춰있을 때 얼마나 기다릴지. 브라우저 연결은 보통 30s~60s.

`timeout server`: 백엔드 서버 쪽 비활성 시간 제한. 서버가 응답을 내려보내지 않고 멈춰있을 때 얼마나 기다릴지. API 처리 시간이 길면 이 값을 넉넉하게 잡아야 한다.

`timeout http-request`: HTTP 요청 헤더 전체가 도착할 때까지 기다리는 시간. Slowloris 공격(헤더를 아주 천천히 보내는 DoS) 방어에 쓰인다.

`timeout http-keep-alive`: Keep-Alive 연결에서 다음 요청이 올 때까지 기다리는 시간. 이 시간이 지나면 연결을 닫는다.

`timeout queue`: 서버가 maxconn으로 꽉 찼을 때 대기열에서 기다리는 시간. 이 시간이 지나도 서버를 배정받지 못하면 503을 반환한다.

## 구간 시각화

```
클라이언트                 HAProxy                  서버
    |                        |                        |
    |---HTTP 요청 전송------->|  ← timeout http-request |
    |                        |---TCP 연결 시도-------->|  ← timeout connect
    |                        |<--연결 완료-------------|
    |                        |---요청 전달------------>|
    |                        |        (서버 처리 중)   |  ← timeout server
    |                        |<--응답-----------------|
    |<--응답 전달------------|
    |     (유휴 상태)         |                        |  ← timeout client / keep-alive
```

## 자주 하는 실수

`timeout server`를 너무 짧게 잡으면 처리가 오래 걸리는 API에서 504 Gateway Timeout이 발생한다. 반대로 너무 길게 잡으면 죽은 서버 연결이 오래 점유된다.

배치 API나 파일 업로드처럼 시간이 오래 걸리는 endpoint가 있다면 별도 backend를 만들어 타임아웃을 다르게 설정하는 것이 낫다.

참고: maxconn.md, queue.md
