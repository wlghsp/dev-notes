# terminal.ts 코드 뜯어보기

`backend/src/terminal.ts` 전체를 한 블록씩 설명한다.
Spring/Java 개념과 비교하면서 읽으면 빠르게 이해된다.

---

## 전체 코드 흐름 먼저

```
브라우저가 WebSocket 연결
    ↓
handleTerminal() 호출
    ↓
1. Docker 컨테이너 생성
2. 컨테이너에 스트림 연결 (attach)
3. 컨테이너 시작
4. 입출력 연결 (브라우저 ↔ 컨테이너)
5. 연결 끊기면 컨테이너 제거
```

---

## 1. import

```typescript
import Docker from 'dockerode'
import type { WebSocket } from 'ws'
```

- `Docker` — dockerode 라이브러리. Docker 데몬(Docker 엔진)과 통신하는 클라이언트.
- `WebSocket` — `ws` 라이브러리의 타입. `import type`이라 런타임엔 포함 안 됨. TypeScript 타입 검사용으로만 쓴다.

Spring 비교:
```java
// Java라면
import com.github.dockerjava.api.DockerClient;
import org.springframework.web.socket.WebSocketSession;
```

---

## 2. Docker 인스턴스 생성

```typescript
const docker = new Docker()
```

- Docker 데몬과 통신할 클라이언트 객체를 만든다.
- 인자 없으면 기본 소켓 경로(`/var/run/docker.sock` 또는 환경변수)를 자동으로 찾는다.
- Colima 사용 시 심볼릭 링크로 경로를 맞춰놨기 때문에 인자 없이도 동작한다.

Spring 비교:
```java
// Java라면
DockerClient docker = DockerClientBuilder.getInstance().build();
```

---

## 3. 함수 선언

```typescript
export async function handleTerminal(socket: WebSocket) {
```

- `export` — 다른 파일에서 import 할 수 있게 공개. `index.ts`에서 이 함수를 가져다 쓴다.
- `async` — 이 함수 안에서 `await`를 쓰겠다는 선언. 함수가 `Promise`를 반환한다.
- `socket: WebSocket` — 브라우저와 연결된 WebSocket 객체. 이걸 통해 브라우저에 데이터를 보내거나 받는다.

Spring 비교:
```java
// Java라면
public void handleTerminal(WebSocketSession session) { ... }
```

---

## 4. 컨테이너 생성

```typescript
const container = await docker.createContainer({
    Image: 'ubuntu',
    Cmd: ['/bin/bash'],
    AttachStdin: true,
    AttachStdout: true,
    AttachStderr: true,
    OpenStdin: true,
    Tty: true,
})
```

`docker run ubuntu /bin/bash`를 코드로 한 것과 같다.

| 옵션 | 의미 |
|------|------|
| `Image: 'ubuntu'` | 사용할 Docker 이미지 |
| `Cmd: ['/bin/bash']` | 컨테이너 시작 시 실행할 명령 |
| `AttachStdin/Stdout/Stderr` | 입력/출력/에러를 연결 허용 |
| `OpenStdin: true` | stdin을 열어둠. 없으면 bash가 바로 종료됨 |
| `Tty: true` | 터미널 모드. 없으면 프롬프트(`$`)와 색상이 안 나옴 |

> `createContainer`는 컨테이너를 **만들기만** 한다. 아직 실행 안 됨.
> `await`로 생성이 완료될 때까지 기다린다.

---

## 5. 스트림 연결 (attach)

```typescript
const stream = await container.attach({
    stream: true,
    stdin: true,
    stdout: true,
    stderr: true,
    hijack: true,
})
```

컨테이너의 입출력 통로(파이프)를 연다.

| 옵션 | 의미 |
|------|------|
| `stream: true` | 스트리밍 모드로 연결 |
| `stdin/stdout/stderr` | 어떤 채널을 연결할지 |
| `hijack: true` | TCP 연결을 직접 제어. 양방향 스트리밍에 필요 |

> **왜 start 전에 attach 하나?**
> 컨테이너가 시작되면서 출력하는 초기 데이터(bash 프롬프트 등)를 놓치지 않으려고.
> attach 먼저 → start 나중 순서가 중요하다.

Spring 비교:
```java
// Java라면 (개념적으로)
InputStream stdout = container.attachStdout();
OutputStream stdin = container.attachStdin();
```

---

## 6. 컨테이너 시작

```typescript
await container.start()
```

`docker start`에 해당한다. 이 시점부터 컨테이너 안의 bash가 실행된다.

---

## 7. 컨테이너 출력 → 브라우저

```typescript
stream.on('data', (chunk: Buffer) => {
    socket.send(chunk)
})
```

- `stream.on('data', ...)` — 컨테이너에서 데이터가 나올 때마다 호출되는 이벤트 핸들러.
- `chunk: Buffer` — 바이너리 데이터 덩어리. 터미널 출력은 텍스트가 아니라 바이너리로 온다.
- `socket.send(chunk)` — 브라우저로 전송.

Spring 비교:
```java
// Java라면 (개념적으로)
stdout.read(buffer);
session.sendMessage(new BinaryMessage(buffer));
```

**이벤트 기반이란?**
Java의 `while (true) { read() }` 루프 방식이 아니다.
데이터가 들어올 때 Node.js가 알아서 콜백을 호출한다. 기다리는 동안 다른 작업을 할 수 있다.

---

## 8. 브라우저 입력 → 컨테이너

```typescript
socket.on('message', (msg: Buffer) => {
    stream.write(msg)
})
```

- `socket.on('message', ...)` — 브라우저에서 키 입력이 올 때마다 호출.
- `stream.write(msg)` — 컨테이너 stdin에 전달. bash가 키 입력으로 받아서 처리.

흐름:
```
사용자 키보드 → 브라우저 xterm.js → WebSocket → socket.on('message') → stream.write() → 컨테이너 bash
```

---

## 9. 연결 종료 시 컨테이너 제거

```typescript
socket.on('close', () => {
    container.stop()
        .then(() => container.remove())
        .catch(() => {})
})
```

- `socket.on('close', ...)` — 브라우저 탭을 닫거나 연결이 끊기면 호출.
- `container.stop()` — 컨테이너 정지 (`docker stop`).
- `.then(() => container.remove())` — 정지 완료 후 컨테이너 삭제 (`docker rm`).
- `.catch(() => {})` — 이미 죽은 컨테이너 등 에러는 무시.

**왜 async/await 대신 .then()인가?**
`socket.on('close', ...)` 콜백은 `async`가 아니다. `async`가 아닌 함수 안에서는 `await`를 쓸 수 없다.
`.then()` 체이닝은 `await` 없이도 순서를 보장하는 방법이다.

```typescript
// 이건 안 됨 — close 콜백이 async가 아님
socket.on('close', () => {
    await container.stop()  // 에러
})

// 이렇게 해야 함
socket.on('close', async () => {
    await container.stop()  // OK — 콜백에 async 붙이면 됨
    await container.remove()
})

// 또는 .then() 체이닝
socket.on('close', () => {
    container.stop()
        .then(() => container.remove())
        .catch(() => {})
})
```

---

## 전체 흐름 다시 보기

```
브라우저 탭 열림
    → WebSocket 연결
    → handleTerminal(socket) 호출
    → docker.createContainer() — 컨테이너 생성
    → container.attach() — 입출력 통로 열기
    → container.start() — 컨테이너 실행, bash 시작
    → stream.on('data') 등록 — 컨테이너 출력 감시 시작
    → socket.on('message') 등록 — 브라우저 입력 감시 시작
    → socket.on('close') 등록 — 종료 감시 시작

[사용자가 키 입력]
    → socket.on('message') 발동
    → stream.write(msg) — 컨테이너로 전달
    → bash가 처리 후 출력
    → stream.on('data') 발동
    → socket.send(chunk) — 브라우저로 전달
    → xterm.js가 화면에 렌더링

[사용자가 탭 닫음]
    → socket.on('close') 발동
    → container.stop() → container.remove()
```

---

## 핵심 개념 정리

| 개념 | 설명 | 관련 문서 |
|------|------|-----------|
| async/await | 비동기 처리 | `glossary/async_await.md` |
| 이벤트 기반 | `.on('이벤트', 콜백)` 패턴 | `glossary/websocket.md` |
| Buffer | 바이너리 데이터 | `glossary/docker_stream.md` |
| stream | 데이터 통로 | `glossary/docker_stream.md` |
