# WebSocket

## 한 줄 요약

한 번 연결하면 서버-클라이언트가 양방향으로 계속 데이터를 주고받을 수 있는 프로토콜.

---

## HTTP와 비교

```
HTTP (일반 REST API)
클라이언트 → 요청 → 서버
클라이언트 ← 응답 ← 서버
(연결 끊김, 다시 요청하려면 새 연결)

WebSocket
클라이언트 ↔ 연결 유지 ↔ 서버
(양방향으로 계속 데이터 전송 가능)
```

터미널은 키 입력할 때마다 서버에 보내고, 서버는 출력을 실시간으로 돌려줘야 합니다. HTTP로는 이걸 구현하기 어렵습니다. WebSocket이 필요한 이유입니다.

---

## Spring과 비교

Spring에서 WebSocket:

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new TerminalHandler(), "/ws/terminal");
    }
}
```

Fastify에서 WebSocket:

```typescript
await fastify.register(websocket)

app.get('/ws/terminal', { websocket: true }, (socket, _req) => {
    socket.on('message', (msg: Buffer) => {
        socket.send(msg)
    })
})
```

Spring은 설정 클래스가 별도로 필요하고, Fastify는 플러그인 등록 한 줄로 끝납니다.

---

## socket 이벤트

WebSocket은 이벤트 기반으로 동작합니다.

```typescript
socket.on('message', (msg) => { ... })  // 메시지 수신 시
socket.on('close', () => { ... })       // 연결 종료 시
socket.on('error', (err) => { ... })    // 에러 발생 시

socket.send(data)  // 데이터 전송
socket.close()     // 연결 종료
```

---

## 이 프로젝트에서 흐름

```
브라우저 xterm.js
    │  키 입력 (binary)
    ▼
WebSocket (ws://localhost:3001/ws/terminal)
    │
    ▼
Fastify socket.on('message')
    │
    ▼
Docker exec stream.write()  → 컨테이너 안 bash
    │
Docker stream.on('data')    ← 컨테이너 출력
    │
    ▼
socket.send(chunk)
    │
    ▼
브라우저 xterm.js term.write()
```
