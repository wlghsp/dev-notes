# Claude Code Architecture — Part 2: Agent 구동 파일, Input Layer, Knowledge Layer

참고: claude-code-architecture-1-overview.md

---

## 에이전트를 움직이는 4가지 파일

Claude Code의 행동은 네 가지 입력으로 제어된다. 각각 로딩 시점과 용도가 다르다.

```
CLAUDE.md   →  항상 로드     (세션 시작 + 매 턴)
SKILL.md    →  필요할 때     (description 매칭 시)
Prompt      →  지금          (사용자 입력 시점)
Hook        →  자동          (라이프사이클 이벤트)
```

### CLAUDE.md — Memory (항상)

프로젝트 영속 컨텍스트. 규칙, 명령, 구조를 매 세션 자동 로드한다.

"항상 읽혀야 할 것"을 여기에 둔다. 예: 코딩 규칙, 브랜치 전략, 커밋 규칙.

주의: 비대해지면 컨텍스트 낭비다. 매 턴 로드되므로 꼭 필요한 것만, 간결하게 유지한다.

### SKILL.md — Skill (필요할 때)

재사용 절차서. description이 작업에 매칭될 때만 본문이 로드된다.

반복되는 how-to를 담는다. 예: PR 리뷰 체크리스트, PDF 폼 채우기 절차.

SKILL.md 구조 예시:
```
---
name: review-pr
description: Reviews PRs for bugs, security, quality. Use when reviewing PRs.
---
# Review Checklist
1. Check security vulns
2. Verify test coverage
```

- `name` → 그대로 /slash-command가 된다
- `description` → 작업에 매칭되면 자동 트리거, 매칭을 위해 작성하는 것 (사람용 설명 아님)
- `body` → 매칭될 때만 컨텍스트에 로드

Anthropic이 오픈 표준으로 공개했고, Cursor/GitHub/JetBrains 등 16+ 도구가 같은 포맷을 채택했다.

### Progressive Disclosure — 3단계 점진적 로딩

스킬이 수백 개여도 컨텍스트가 안 부풀리는 이유가 여기 있다.

- Level 1: 이름 + 설명만 (30-50 토큰). 에이전트가 항상 읽는 메타데이터
- Level 2: SKILL.md 본문. 작업과 매칭되면 전체 지시를 로드. 5,000 토큰 이하 권장
- Level 3: scripts/, references/ — 본문이 요구할 때만 온디맨드 로드

### 스킬 디렉터리 구조

```
.claude/skills/review-pr/
  SKILL.md          (필수: 지시)
  scripts/          (실행 코드)
    lint.sh
    scan.py
  references/       (문서, 선택)
    api-docs.md
  assets/           (템플릿, 선택)
    report-template.md
```

핵심 프론트매터 필드:
- `name` — /slash-command로 직접 호출
- `description` — 자동 매칭 트리거 (가장 중요)
- `allowed_tools` — 도구 접근 권한 제어
- `agent` — 서브에이전트로 실행 여부
- `hooks` — 라이프사이클 콜백 연결

SKILL.md 500줄 이하 권장, 지시 5,000 토큰 이하 권장.

### Prompt — 지금 (일회성)

지금의 일회성 지시. 눈앞의 작업 하나를 처리하는 단발성 요청에 쓴다.

예: "getUser를 리팩터링해줘". 반복되는 패턴이라면 SKILL.md로 옮겨라.

주의: 모호한 지시는 모호한 결과를 낳는다. 목표, 제약, 완료 기준을 분명히 써라.

### Hook — 자동 (이벤트 트리거)

settings.json의 결정적 코드. 이벤트에 자동 실행된다.

Claude Code의 특정 시점에 실행되는 셸 명령 또는 HTTP 핸들러다. 이벤트가 발생하면 JSON 컨텍스트가 전달되고, 핸들러는 검사 후 허용/차단 결정을 반환할 수 있다.

라이프사이클 이벤트:
- 세션 1회: SessionStart → SessionEnd
- 턴 1회: UserPromptSubmit → Stop → StopFailure
- 도구 호출마다: PreToolUse → PostToolUse → PostToolUseFailure
- 그 외: SubagentStart, SubagentStop, PermissionRequest, Notification, PreCompact, PostCompact

용도: lint 실행, 스크립트 차단, 로깅.

주의: 잘못된 exit code가 에이전트를 멈출 수 있다. 격리 환경에서 먼저 검증 후 실제 적용하라.

---

## Skills vs Configs vs MCP — 언제 뭘 쓰나

세 가지는 상호 보완적이다. 겹치지 않는다.

Skills (SKILL.md):
- 온디맨드 전문성. 트리거 시 로드
- 반복 워크플로에 적합
- 16+ 도구에서 같은 오픈 포맷 사용

Configs (CLAUDE.md):
- 항상 켜진 컨텍스트. 매 세션 로드
- 프로젝트 규칙, 명명 규칙, 구조
- 단일 도구 전용

MCP:
- 외부 도구/데이터. 시작 시 연결
- API, DB, 외부 서비스 연동
- 오픈 프로토콜, 3000+ 생태계

---

## Input Layer — 인터페이스와 실행 환경

에이전틱 루프와 도구, 기능은 어디서나 동일하다. 달라지는 것은 실행 위치와 상호작용 방식이다.

실행 환경:
- Local: 내 머신에서 실행. 파일/도구 전체 접근 (기본값)
- Cloud: Anthropic 관리 VM에서 실행. 작업 오프로드
- Remote Control: 브라우저로 제어하되 내 머신에서 로컬 실행

인터페이스: 터미널(CLI), 데스크톱 앱, VS Code/JetBrains, claude.ai/code(웹), Remote Control, Slack, CI-CD(GitHub Actions), tag @claude(PR)

세션은 로컬 JSONL 파일에 영구 저장된다.

### Sessions & Permissions

세션 연속성 (Resume / Fork):
- Resume: 같은 세션 ID에 메시지 추가. 대화 이어가기
- Fork (`--fork-session`): 새 ID로 히스토리 복사. 분기 실험

Checkpoints:
- 편집 전 파일 스냅샷 자동 저장
- Esc 두 번으로 되감기
- git과 별개. 파일 변경만 커버 (외부 부수효과는 체크포인트 불가)

권한 모드 (Shift+Tab으로 순환):
- Default: 파일 편집/셸 명령 전 매번 확인
- Auto-accept edits: 편집/일반 FS 명령 자동, 그 외 확인
- Plan mode: 읽기 전용 도구만, 계획 후 승인
- Auto mode: 백그라운드 안전성 검사 (research preview)

`.claude/settings.json`으로 특정 명령 사전 허용. 조직에서 개인까지 정책 스코프 계층 구성 가능.

---

## Knowledge Layer — 컨텍스트와 메모리

### 컨텍스트 윈도우에 들어오는 것들

Context Window 안에는:
- 대화 히스토리
- 파일 내용
- 명령 출력
- CLAUDE.md
- Auto Memory (MEMORY.md 첫 200줄/25KB)
- 로드된 Skills
- 시스템 지시

영구적으로 적용할 규칙은 대화가 아니라 CLAUDE.md에 둬야 한다.

CLAUDE.md는 하위 디렉터리 우선 적용된다. Subagents는 독립 컨텍스트에서 실행하고 요약만 반환한다.

### Context Window 관리 — 자동 컴팩션

컨텍스트가 한계에 가까워지면 Claude Code가 자동으로 관리한다.

처리 순서:
1. 컨텍스트 채워짐 (한계 근접)
2. 오래된 tool output 우선 제거
3. 대화 요약 (필요 시 compaction)
4. 핵심 보존, 작업 지속 — 요청/코드는 유지, 초기 상세 지시는 유실 가능

제어 방법:
- Compact Instructions: CLAUDE.md 섹션으로 보존 내용 지정
- `/compact focus` 명령: 요약 초점 직접 지정
- `/context` 명령: 현재 사용량 확인
- MCP 도구는 deferred. tool search로 온디맨드. 이름만 소비

참고: claude-code-architecture-3-execution.md
