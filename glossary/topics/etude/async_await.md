# async/await — 비동기 처리

## 한 줄 요약

작업이 끝날 때까지 기다리되, 그 동안 다른 일을 할 수 있는 처리 방식.

---

## Spring과 비교

Spring(Java)에서는 보통 이렇게 씁니다:

```java
// 동기 — 응답 올 때까지 스레드가 블로킹됨
Container container = docker.createContainer(options);
container.start();
```

Node.js에서는:

```typescript
// 비동기 — 기다리는 동안 다른 요청 처리 가능
const container = await docker.createContainer(options)
await container.start()
```

겉모양은 비슷한데 동작 방식이 다릅니다.

- **Java(동기)**: 컨테이너 생성되는 동안 해당 스레드는 멈춰서 기다림. 요청이 100개면 스레드 100개 필요.
- **Node.js(비동기)**: 컨테이너 생성 요청만 보내고 기다리는 동안 다른 요청 처리. 스레드 1개로 요청 100개 처리 가능.

---

## Promise란

`await` 앞에 붙는 값은 `Promise`입니다. "나중에 결과를 줄게"라는 약속 객체입니다.

```typescript
// 이 두 개는 같은 코드
const container = await docker.createContainer(options)

docker.createContainer(options).then((container) => {
  // container 사용
})
```

`await`는 `.then()` 체이닝을 읽기 쉽게 바꾼 문법입니다.

---

## async 함수

`await`를 쓰려면 함수에 `async`를 붙여야 합니다.

```typescript
// async 없으면 await 사용 불가
async function handleTerminal(socket: WebSocket) {
  const container = await docker.createContainer(...)  // OK
}
```

`async` 함수는 항상 `Promise`를 반환합니다.

---

## 이 프로젝트에서 나오는 곳

```typescript
// terminal.ts — 컨테이너 생성/시작/exec 전부 await
export async function handleTerminal(socket: WebSocket) {
  const container = await docker.createContainer({ ... })
  await container.start()
  const exec = await container.exec({ ... })
  const stream = await exec.start({ ... })
}

// index.ts — Fastify 플러그인 등록도 await
await fastify.register(websocket)
await fastify.listen({ port: 3001 })
```
