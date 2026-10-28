# 외부 접속 원리

같은 네트워크(같은 공유기)에 있는 다른 PC가 내 PC에서 돌아가는 서버에 접속하는 원리.

---

## 왜 기본값은 localhost인가

서버 프로세스를 실행할 때 어느 네트워크 인터페이스에서 요청을 받을지 지정할 수 있다.

| 바인딩 주소 | 의미 |
|---|---|
| `127.0.0.1` (localhost) | 내 PC 내부에서만 접근 가능 |
| `0.0.0.0` | 내 PC에 연결된 모든 네트워크 인터페이스에서 접근 가능 |

Vite나 Fastify는 기본값이 `127.0.0.1`이라 외부에서 접속할 수 없다.
`host: '0.0.0.0'`으로 바꾸면 외부 요청도 받아들인다.

```
다른 PC ──(공유기)──▶ 192.168.0.117:5173 (내 PC)
                            ↓
                    Vite가 0.0.0.0으로 바인딩되어 있어 수신 가능
```

---

## window.location.hostname

브라우저에서 `window.location.hostname`은 현재 페이지를 열 때 사용한 호스트 주소를 반환한다.

| 접속 URL | window.location.hostname |
|---|---|
| `http://localhost:5173` | `localhost` |
| `http://192.168.0.117:5173` | `192.168.0.117` |

API 주소를 `localhost:3001`로 하드코딩하면, 다른 PC의 브라우저는 **자기 자신의** `localhost:3001`로 요청을 보내 실패한다.

`window.location.hostname`을 쓰면 브라우저가 자동으로 서버 IP를 사용하므로 어디서 접속하든 올바른 주소로 요청이 간다.

```typescript
// 하드코딩 — 다른 PC에서 접속 시 자기 자신의 localhost로 요청
const BASE = 'http://localhost:3001'

// 동적 — 접속한 주소 기준으로 백엔드 주소 결정
const BASE = `http://${window.location.hostname}:3001`
```

WebSocket도 동일하다:
```typescript
`ws://${window.location.hostname}:3001/ws/terminal?...`
```

---

## CORS

브라우저는 다른 출처(origin)로 요청을 보낼 때 서버가 허용했는지 먼저 확인한다.

`origin: 'http://localhost:5173'`으로 고정하면 `http://192.168.0.117:5173`에서 오는 요청은 CORS 오류로 막힌다.

`origin: true`로 설정하면 요청 헤더의 Origin을 그대로 허용해 외부 접속도 통과한다. 내부망 테스트 환경에서는 충분하다.

---

## 정리

| 변경 | 이유 |
|---|---|
| Vite `host: '0.0.0.0'` | 외부 PC의 요청을 수신하기 위해 |
| `window.location.hostname` | 브라우저가 올바른 서버 주소로 요청하도록 |
| CORS `origin: true` | 외부 출처 요청을 허용하기 위해 |
