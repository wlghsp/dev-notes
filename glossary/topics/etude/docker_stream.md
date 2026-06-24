# Docker 컨테이너 스트리밍

## 한 줄 요약

dockerode로 컨테이너를 만들고, exec으로 bash를 붙이고, stream으로 입출력을 연결하는 흐름.

---

## 전체 흐름

```
1. createContainer — 컨테이너 생성 (아직 실행 안 됨)
2. container.start() — 컨테이너 실행
3. container.exec() — 실행 중인 컨테이너에 명령 붙이기
4. exec.start() — 실제로 실행 + 스트림 연결
5. stream — 입출력 통로
```

---

## 코드로 보기

```typescript
// 1. 컨테이너 생성
const container = await docker.createContainer({
  Image: 'ubuntu',      // 사용할 Docker 이미지
  Cmd: ['/bin/bash'],   // 시작 명령
  AttachStdin: true,    // 입력 연결
  AttachStdout: true,   // 출력 연결
  AttachStderr: true,   // 에러 출력 연결
  OpenStdin: true,      // stdin 열어두기
  Tty: true,            // 터미널 모드 (프롬프트, 색상 등 지원)
})

// 2. 컨테이너 시작
await container.start()

// 3. exec — 실행 중인 컨테이너에 bash 붙이기
const exec = await container.exec({
  Cmd: ['/bin/bash'],
  AttachStdin: true,
  AttachStdout: true,
  AttachStderr: true,
  Tty: true,
})

// 4. 스트림 연결
const stream = await exec.start({ hijack: true, stdin: true })

// 5. 입출력 연결
stream.on('data', (chunk: Buffer) => {
  socket.send(chunk)       // 컨테이너 출력 → 브라우저
})
socket.on('message', (msg: Buffer) => {
  stream.write(msg)        // 브라우저 입력 → 컨테이너
})
```

---

## Tty가 뭔가

`Tty: true` 없으면 bash가 프롬프트(`$`)를 안 보여줍니다. 터미널처럼 동작하게 하는 옵션입니다.

---

## createContainer vs exec 차이

| | createContainer | exec |
|---|---|---|
| 언제 | 컨테이너 처음 만들 때 | 이미 실행 중인 컨테이너에 명령 추가할 때 |
| 비유 | 서버 부팅 | 부팅된 서버에 SSH 접속 |

이 프로젝트에서는 컨테이너를 만들고 바로 exec으로 bash를 붙입니다.

---

## 연결 종료 처리

```typescript
socket.on('close', () => {
  container.stop()
    .then(() => container.remove())
    .catch(() => {})  // 이미 종료된 경우 무시
})
```

브라우저 탭을 닫으면 WebSocket이 끊기고 → `close` 이벤트 → 컨테이너 정지 및 삭제.
`await` 대신 `.then()` 체이닝을 쓰는 이유: `close` 콜백은 `async` 함수가 아니라 `await`를 쓸 수 없습니다.
