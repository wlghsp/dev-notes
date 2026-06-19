# Claude Code Architecture — Part 3: Execution, Multi-Agent, Observability, 프리미티브 선택

참고: claude-code-architecture-1-overview.md, claude-code-architecture-2-inputs.md

---

## Execution Layer — 도구 실행 파이프라인

Take Action 단계에서 모델은 구조화된 도구 호출을 내보내고, 핸들러가 이를 해석해 실행한다.

```
Model → Tool Dispatch → Execute → Result
tool_use 블록 생성  도구별 핸들러로  파일/셸/웹/git  결과를 컨텍스트로
                   라우팅(typed    수행             환원
                   registry)
        ↑_____________결과 피드백 (루프 반복)___________________________|
```

### Prompt Cache

- 5분 / 1시간 TTL
- 안정적 프리픽스 재사용
- 캐시 읽기 비용 약 10%
- CLAUDE.md처럼 매 턴 반복되는 내용이 여기서 절감된다

### Streaming Runtime

- 실시간 출력
- 독립적 도구 호출은 병렬 실행

---

## Multi-Agent Layer — 서브에이전트와 Worktrees

### 서브에이전트 — 격리된 추론과 위임

메인 에이전트가 계획과 통합을 맡고, 특화된 서브에이전트가 각자 자체 컨텍스트에서 작업한다. 끝나면 요약만 반환해 메인 컨텍스트를 오염시키지 않는다.

```
Main Agent (계획 + 통합)
    ↓ spawn (자체 fresh 컨텍스트)
Backend 구현 | Frontend 구현 | Test 실행 | Review 코드리뷰
    ↑ Return Summary (완료 시 요약만 반환, 작업 내용은 메인 컨텍스트에서 격리)
```

핵심 특성:
- 격리 샌드박스: 서브에이전트 자체 컨텍스트
- 병렬 실행: 동시 실행, 실패 격리
- `/agents` 명령으로 서브에이전트 구성

### Worktrees — 병렬 세션

git worktree로 병렬 세션을 동시 실행한다. 원본을 유지하면서 분기해 별별 세션을 독립적으로 실행하는 방식이다.

---

## Observability Layer — Hooks

Hooks는 Claude Code의 특정 시점에 실행되는 셸 명령 또는 HTTP 핸들러다. 이벤트가 발생하면 JSON 컨텍스트가 전달되고, 핸들러는 검사 후 허용/차단 결정을 반환할 수 있다.

라이프사이클 이벤트 (12가지):

세션 수준 (세션 1회):
- SessionStart → SessionEnd

턴 수준 (턴 1회):
- UserPromptSubmit → Stop → StopFailure

도구 수준 (도구 호출마다):
- PreToolUse → PostToolUse → PostToolUseFailure

그 외:
- SubagentStart / SubagentStop
- PermissionRequest
- Notification
- PreCompact / PostCompact

PreToolUse: 실행 전 검증/차단 용도 (예: 보안 스캔)
PostToolUse: 포맷/테스트/로깅 용도 (예: lint 자동 실행)

Background 작업: non-blocking으로 관찰. 루프를 막지 않으면서 모니터링 가능.

---

## Integration Layer — MCP

MCP(Model Context Protocol)는 AI 모델이 외부 도구와 데이터 소스에 접근하는 표준 프로토콜이다. MCP 서버가 도구를 노출하면 Claude(MCP 클라이언트)가 이를 자동 발견해 호출한다.

```
Claude Code          MCP          MCP Servers (도구 노출)
(MCP Client)  ↔  Protocol  ↔  Filesystem | Git/GitHub | Database
                auto-discover    Sentry    | Custom     | 3000+
```

도구 정의는 deferred. tool search로 온디맨드 로드 (`/mcp`로 서버별 컨텍스트 비용 확인).

연결 가능한 외부 서버 예시:
- Filesystem (파일 접근)
- Git / GitHub (버전관리/PR)
- Database (쿼리/스키마)
- Sentry (모니터링)
- Custom (사내 도구)
- 3000+ 생태계 통합

---

## 프리미티브 선택 — 언제 뭘 쓰나

### 결정 트리

```
지식 vs 행동?
├── 지식: 항상 vs 조건부?
│   ├── 항상 → CLAUDE.md (매 세션 로드, 별도 프리미티브 아님)
│   └── 조건부 → SKILLS (조건부 지식)
└── 행동: 내부 vs 외부?
    ├── 내부: 부모 vs 격리?
    │   ├── 부모 컨텍스트 내 → 표준 도구 (별도 프리미티브 아님)
    │   └── 격리 → SUBAGENTS (격리 추론)
    └── 외부: 모델 결정 vs 결정적?
        ├── 모델이 호출 시점 결정 → MCP (외부 시스템)
        └── 결정적 강제 → HOOKS (이벤트 자동화)
```

### 4가지 프리미티브 특성 비교

Skills:
- 목적: 조건부 작업 지식
- 트리거: 모델이 컨텍스트로 판단
- 실행 컨텍스트: 같은 대화 컨텍스트
- 실패 영향: 낮음 (오용 시 환각)
- 예시: pr-review-checklist

Subagents:
- 목적: 격리된 추론 / 위임
- 트리거: 모델이 명시적 위임
- 실행 컨텍스트: 격리 샌드박스
- 실패 영향: 중간 (격리 누수/부풀림)
- 예시: code-auditor

MCP:
- 목적: 모델이 호출하는 외부 시스템
- 트리거: 모델이 호출 시점 결정
- 실행 컨텍스트: 외부 프로세스
- 실패 영향: 높음 (네트워크/인증/장애)
- 예시: github, postgres

Hooks:
- 목적: 결정적 강제 / 자동화
- 트리거: 이벤트 (PreToolUse 등)
- 실행 컨텍스트: 로컬 훅 러너
- 실패 영향: 높음 (흐름 차단/손상)
- 예시: pre-tool-security-scan

### 설계 원칙

가장 단순한 프리미티브부터 시작: Skills → Subagents → MCP → Hooks 순.

Skills — 부풀림 주의:
- 거대 문서 덤프는 컨텍스트 부풀림
- 작게, 정밀하게, 조건부로 유지

Subagents — 격리/위임 절제:
- 느슨한 격리와 과도한 위임 회피
- 명확한 계약, 최소 입출력, 강한 스키마

MCP — 외부 신뢰성:
- 불안정 외부와 인증 실패, 도구 환각 주의
- 타임아웃, 재시도, 좋은 설명으로 보완

Hooks — 안전한 자동화:
- 치명적 동작 차단과 무한 루프 주의
- 안전 기본값, 멱등성, 철저한 테스트

---

## End-to-End 전체 흐름

요청이 들어와서 검증된 결과로 나가기까지:

```
1 Prompt      →  2 Permission   →  3 Gather Context  →
(Input Layer)    (Input Layer)     (Knowledge Layer)
사용자 요청 입력  행동 전 권한 확인   CLAUDE.md/Skills/메모리

4 Take Action →  5 Verify       →  6 Task Result
(Execution)      (Core Loop)       (Output)
도구/MCP/서브에이전트 테스트로 결과 검증  검증된 산출물 + 메모리
```

전 과정에서 Hooks가 라이프사이클 이벤트로 관찰/검증/차단 (PreToolUse / PostToolUse / Stop).
미완이면 다음 단계 결정하며 루프 반복 (메모리 갱신).

---

## 핵심 요약 — 여덟 개 레이어, 하나의 루프

- Core: Agentic Loop — 수집-행동-검증을 적응적으로 반복
- Input: 진입/세션/권한 — 어디서나 같은 루프, 명시적 권한 통제
- Knowledge: 컨텍스트/메모리 — CLAUDE.md/메모리/Skills + 자동 압축
- Execution: 도구 실행/캐시 — tool_use 디스패치, 캐시/스트리밍 최적화
- Multi-Agent: 서브에이전트 — 위임과 컨텍스트 격리, 병렬/실패 격리
- Observability: Hooks — 라이프사이클 이벤트로 결정론적 통제
- Integration: MCP — 표준 프로토콜로 외부 도구/데이터 연결
- Output: Task Result — 검증된 산출물 + 갱신된 메모리

1 loop (수집-행동-검증) + 2 축 (모델 + 도구) + 4 확장 프리미티브 (Skills/MCP/Hooks/Subagents) + 사람 통제 (권한 + 체크포인트).
