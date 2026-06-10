# Etude 개발 계획

## 확정 스택

- 프론트: React (TypeScript)
- 백엔드: Node.js + Fastify (TypeScript)
- DB: MariaDB
- 터미널: xterm.js (프론트) + WebSocket (Fastify)
- Docker 제어: dockerode
- 샌드박스: Docker 컨테이너 (퀘스트마다 독립 환경)
- 배포: 프론트 Vercel / 백엔드 Railway 무료 티어

TypeScript로 프론트와 백엔드 언어가 통일된다.

---

## MVP 범위

- 브라우저 터미널 실행 (xterm.js + WebSocket + Docker 샌드박스)
- 퀘스트 1~2개 하드코딩 (DB 없이 시작)
- 채점 — 특정 명령어 실행 결과를 서버가 확인해서 성공/실패 판정
- 로그인 없음, 리더보드 없음, AI 퀘스트 생성 없음

핵심 검증 목표: "브라우저에서 터미널을 열고, 명령어를 실행하고, 채점까지 되는가"

---

## 아키텍처

```
브라우저 (React)
    ├── xterm.js 터미널 UI
    └── WebSocket 연결 → Fastify

Fastify 서버 (Node.js + TypeScript)
    ├── WebSocket 엔드포인트 — 터미널 입출력 스트리밍
    ├── Docker 컨테이너 생성/제거 (dockerode)
    ├── 퀘스트 API — 퀘스트 지문, 채점 기준 제공
    └── 채점 API — 컨테이너 상태 확인 후 성공/실패 반환

Docker 컨테이너 (샌드박스)
    └── 사용자별 격리된 Linux 환경
        명령어 실행 결과가 여기서 나옴
        퀘스트 종료 시 컨테이너 제거
```

---

## Phase 1 — 터미널 샌드박스 단독 작동

목표: 브라우저에서 터미널 열고 명령어 실행되는 것만 확인

### 백엔드 (Fastify)
- Fastify 프로젝트 초기화 (TypeScript)
- dockerode로 컨테이너 생성/실행/제거 기본 구현
- `@fastify/websocket` 플러그인으로 WebSocket 엔드포인트 구현
- 입력 받아서 컨테이너에 전달, 출력 브라우저로 스트리밍

```typescript
// 핵심 흐름
fastify.get('/ws/terminal', { websocket: true }, (socket, req) => {
  const container = await docker.createContainer({ Image: 'ubuntu' })
  // 입력 → 컨테이너 exec → 출력 → 브라우저
})
```

### 프론트 (React + TypeScript)
- React 프로젝트 초기화 (Vite + TypeScript)
- xterm.js 설치 및 터미널 컴포넌트 구현
- WebSocket 연결해서 입출력 연동

### 검증
- 브라우저 터미널에서 `ls`, `pwd`, `echo hello` 실행되면 Phase 1 완료

---

## Phase 2 — 퀘스트 붙이기

목표: 퀘스트 지문 + 터미널 레이아웃 + 채점

### 화면 구성
```
┌─────────────────┬─────────────────┐
│   퀘스트 지문    │    터미널        │
│                 │                 │
│ 미션: /tmp에    │ $ mkdir /tmp/   │
│ hello 디렉토리  │ hello           │
│ 를 만드세요     │ $               │
│                 │                 │
│ [채점하기]      │                 │
└─────────────────┴─────────────────┘
```

### 백엔드
- 퀘스트 하드코딩 (TypeScript 객체)
- 채점 API — 컨테이너 안에서 조건 확인 후 성공/실패 반환

```typescript
// 채점 예시: /tmp/hello 디렉토리 존재 여부 확인
async function gradeQuest(container: Container, questId: string) {
  const exec = await container.exec({ Cmd: ['test', '-d', '/tmp/hello'] })
  const result = await exec.start({})
  return result.StatusCode === 0
}
```

### 퀘스트 예시 2개
1. 리눅스 기초 — `/tmp/hello` 디렉토리 만들기
2. 파일 조작 — 특정 파일에 특정 내용 쓰기

### 검증
- 퀘스트 읽고 터미널에서 풀고 채점까지 한 사이클 완료되면 Phase 2 완료

---

## Phase 3 — 로컬에서 직접 써보기

목표: 실제로 써보면서 UX 문제 발견 및 수정

- 퀘스트 난이도/지문이 명확한지
- 터미널 반응 속도
- 채점 피드백이 충분한지
- 컨테이너 생성/제거 타이밍 문제 없는지

여기서 나온 피드백으로 방향 조정 후 다음 단계 결정.

---

## 프로젝트 구조

```
etude/
├── frontend/                   # React + TypeScript
│   ├── src/
│   │   ├── components/
│   │   │   ├── Terminal.tsx    # xterm.js 터미널
│   │   │   └── QuestPanel.tsx  # 퀘스트 지문 + 채점 버튼
│   │   └── App.tsx
│   └── package.json
│
└── backend/                    # Fastify + TypeScript
    ├── src/
    │   ├── index.ts            # Fastify 앱 진입점
    │   ├── terminal.ts         # WebSocket + Docker 제어
    │   └── quest.ts            # 퀘스트 데이터 + 채점 로직
    ├── package.json
    └── tsconfig.json
```

---

## 주요 라이브러리

### 프론트
- xterm — 브라우저 터미널 UI
- @xterm/addon-fit — 터미널 크기 자동 조절

### 백엔드
- fastify — 웹 프레임워크
- @fastify/websocket — WebSocket 플러그인
- dockerode — Docker 컨테이너 제어
- @types/dockerode — TypeScript 타입 지원

---

## 미결 사항

- [ ] Docker 이미지 선택 — ubuntu 기본 이미지 vs 커스텀 이미지
- [ ] 컨테이너 생명주기 — 세션 종료 시 자동 제거 타이밍
- [ ] 채점 방식 구체화 — 명령어 실행 결과 비교 vs 파일 상태 확인
- [ ] MariaDB 도입 시점 — Phase 2까지는 하드코딩, Phase 3 이후 DB 연동
