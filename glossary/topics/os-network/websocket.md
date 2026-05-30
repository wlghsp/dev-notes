# WebSocket

HTTP 위에서 동작하는 **양방향 실시간 통신 프로토콜**.

이름에 Socket이 들어가지만 Unix Domain Socket이나 TCP 소켓과는 다른 개념이다.
WebSocket은 애플리케이션 계층 프로토콜이고, TCP 소켓은 OS 수준의 통신 추상화다. 참고: socket.md

## 왜 WebSocket이 필요한가

HTTP는 클라이언트가 요청해야만 서버가 응답할 수 있다. 서버가 먼저 데이터를 보내는 게 불가능하다.

채팅, 실시간 알림, 주식 시세 같은 기능은 서버가 먼저 데이터를 밀어줘야 한다.
HTTP만으로는 클라이언트가 계속 폴링(주기적으로 요청)해야 하는데, 이건 비효율적이다.

WebSocket은 연결을 한 번 맺으면 서버와 클라이언트가 언제든 먼저 데이터를 보낼 수 있다.

## HTTP에서 WebSocket으로: 업그레이드 핸드셰이크

WebSocket 연결은 HTTP 요청으로 시작한다. 연결 도중에 프로토콜을 바꾸는 방식이다.

```
클라이언트 → 서버  (HTTP 요청)
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZQ==

서버 → 클라이언트  (HTTP 응답)
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

101 Switching Protocols를 받는 순간부터 HTTP 연결이 WebSocket 연결로 전환된다.
이후부터는 HTTP가 아니라 WebSocket 프레임으로 통신한다.

## 연결 이후

한 번 업그레이드되면 TCP 연결은 유지된 채로 양방향 통신이 가능해진다.

```
클라이언트                서버
    |                     |
    |<---- 메시지 ---------|   서버가 먼저 보낼 수 있음
    |                     |
    |----- 메시지 -------->|   클라이언트도 언제든 보낼 수 있음
    |                     |
    |        ...          |   연결 유지
```

연결을 끊을 때는 Close 프레임을 보내고, 그 다음 TCP 4-way handshake로 종료된다. 참고: tcp.md

## HTTP와 비교

HTTP는 요청-응답 구조다. 클라이언트가 요청해야 서버가 응답한다. 연결은 응답 후 끊긴다(HTTP/1.1은 Keep-Alive로 재사용하지만 구조는 같다).

WebSocket은 연결을 유지한 채로 양쪽이 자유롭게 메시지를 주고받는다. 요청-응답 구조가 없다.

## 계층 위치

```
WebSocket 프로토콜   (애플리케이션 계층)
HTTP               (업그레이드 핸드셰이크에만 사용)
TLS                (wss:// 일 때)
TCP                (전송 계층)
```

ws://는 평문, wss://는 TLS 암호화 적용이다.

## 사용 사례

서버가 먼저 데이터를 보내야 하거나, 지연이 낮아야 하는 상황에 쓴다.

- 채팅
- 실시간 알림
- 주식/코인 시세
- 멀티플레이어 게임
- 협업 편집 (Google Docs 같은 것)
