# 터미널 전체 흐름

## 한 줄 요약

브라우저에서 퀘스트를 선택하면 백엔드가 Docker 컨테이너를 띄우고, WebSocket으로 터미널 입출력을 연결한다.

---

## 전체 흐름

```
1. 사용자가 퀘스트 세트 선택
      ↓
2. App.tsx — sandboxType, questId 상태 업데이트
      ↓
3. Terminal.tsx — WebSocket 연결 요청
   ws://localhost:3001/ws/terminal?sandboxType=linux&questId=3
      ↓
4. index.ts — 쿼리 파라미터 파싱 → handleTerminal 호출
      ↓
5. terminal.ts — sandbox 설정 조회 → 컨테이너 생성/시작
      ↓
6. setup_cmd 실행 (퀘스트 초기 환경 세팅)
      ↓
7. 백엔드 → 프론트로 { type: 'connected' } 전송
      ↓
8. Terminal.tsx — 로딩 해제, xterm.js 터미널 표시
      ↓
9. 사용자 입력 → WebSocket → 컨테이너 stdin
   컨테이너 출력 → WebSocket → xterm.js 화면
```

---

## 코드로 보기

**프론트 (Terminal.tsx)**
```typescript
// WebSocket 연결
const ws = new WebSocket(
    `ws://localhost:3001/ws/terminal?sandboxType=${sandboxType}&questId=${questId}`
)

// 서버에서 connected 오면 로딩 해제
ws.onmessage = (e) => {
    const msg = JSON.parse(e.data)
    if (msg.type === 'connected') setLoading(false)
}

// 키 입력 → 컨테이너로
term.onData((data) => ws.send(data))
```

**백엔드 (index.ts)**
```typescript
app.get('/ws/terminal', { websocket: true }, (socket, req) => {
    const params = new URL(req.url, 'http://localhost').searchParams
    const sandboxType = params.get('sandboxType') ?? 'linux'
    const questId = params.get('questId') ? Number(params.get('questId')) : null
    handleTerminal(socket, docker, sandboxType, questId)
})
```

**백엔드 (terminal.ts)**
```typescript
// sandboxType에 따라 분기
if (sandboxType === 'docker') {
    await handleDockerTerminal(...)   // DinD — exec 방식
} else {
    await handleDefaultTerminal(...)  // linux/linux-ssh — attach 방식
}
```

---

## sandboxType별 차이

| sandboxType | 이미지 | 연결 방식 |
|---|---|---|
| `linux` | ubuntu | attach |
| `linux-ssh` | etude-ssh | attach |
| `docker` | docker:dind | exec (dockerd가 attach 점유) |

---

## 로딩 UI가 있는 이유

컨테이너 생성 + setup_cmd 실행에 시간이 걸린다.
특히 DinD는 dockerd가 뜰 때까지 대기(waitForDocker)가 추가된다.
`connected` 메시지가 오기 전까지 "환경 준비 중..." 을 보여주고, 터미널은 숨긴다.
