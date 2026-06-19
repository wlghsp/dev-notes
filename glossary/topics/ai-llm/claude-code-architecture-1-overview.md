# Claude Code Architecture — Part 1: 개요, 전체 구조, Agentic Loop, Models & Tools

출처: AWS Korea Principal SA Choi WooHyung의 발표 자료 (2026)

---

## Claude Code란

Claude Code는 Claude 모델을 감싸는 에이전트 하네스(Agentic Harness)다.

단순히 LLM에게 질문하고 텍스트를 받는 구조가 아니다. 모델에 도구(tools), 컨텍스트 관리, 실행 환경을 얹어서 실제로 코드를 작성하고 검증하는 코딩 에이전트로 만든 것이다.

핵심 특성 네 가지:

1. Terminal-native — CLI에서 직접 동작. IDE/CI-CD/웹으로도 확장 가능
2. Codebase-aware — 프로젝트 전체 파일, git 상태, 의존 관계를 에이전틱 검색으로 파악
3. Tool-driven — 파일 편집, 셸 실행, 웹 검색 등 도구로 실제 작업을 수행하고 반복
4. Human-in-loop — 수정/실행 전 명시적 권한 확인, 체크포인트로 모든 변경 되돌리기 가능

숫자로 보면: 에이전틱 루프 3 phase, 내장 도구 카테고리 5종, 확장 레이어 4개, 실행 환경 3개.

---

## 전체 아키텍처 — 8개 레이어

모든 것은 Master Agent Loop를 중심으로 돌아간다. 레이어별로 역할이 나뉜다.

```
                        INPUT LAYER
               UI(CLI/IDE) | Session Mgr | Permission
                           |
    OBSERVABILITY          ↓            KNOWLEDGE LAYER
    Hooks / Background → [Master Agent Loop] ← CLAUDE.md / Skills
                           Gather-Act-Verify    Auto Memory / Context Win
    MULTI-AGENT            |
    Subagents / Worktrees  ↓ execute
                     EXECUTION LAYER → INTEGRATION → OUTPUT LAYER
                Tool Dispatch / Cache  MCP Runtime   Task Result
                Streaming              Ext Servers
```

레이어 목록:

- Input Layer: 사용자가 요청을 넣는 진입점. UI, 세션 관리, 권한 처리
- Knowledge Layer: 루프에 지식을 공급. CLAUDE.md, Auto Memory, Skills, Context Window
- Execution Layer: 도구 실행 파이프라인. Tool Dispatch, Prompt Cache, Streaming
- Integration Layer: 외부 연결. MCP Runtime, External Servers
- Observability Layer: 라이프사이클 관찰/통제. Hooks, Background 작업
- Multi-Agent Layer: 서브에이전트, Worktrees로 병렬/격리 실행
- Output Layer: 최종 결과. 검증된 산출물 + 메모리 갱신

---

## The Agentic Loop — 3단계 핵심 루프

사용자 프롬프트가 들어오면 Claude는 세 단계를 반복한다.

```
Prompt → Gather Context → Take Action → Verify Results → Complete
           파일/코드 검색    편집/명령/도구    테스트/타입체크     작업 완료
                ↑___________________학습한 내용으로 다음 단계 결정 (반복)____|
```

- Gather: 맥락을 파악한다. 파일 읽기, 코드 검색
- Act: 도구로 행동한다. 파일 편집, 명령 실행, 도구 호출
- Verify: 결과를 확인한다. 테스트 실행, 타입체크
- Esc: 언제든 중단/재지시 가능

작업 성격에 따라 루프가 다르게 동작한다:

- 질문형 작업 → Gather 단계만으로 응답
- 버그 수정 → 세 단계를 여러 번 반복
- 리팩터링 → Verify 단계에 더 많은 비중

---

## Models & Tools — 루프를 움직이는 두 축

### Models (추론 엔진)

- Sonnet: 대부분의 코딩 작업 담당
- Opus: 복잡한 아키텍처 추론, 설계 담당
- `/model` 명령으로 세션 중 전환 가능

도구가 없으면 텍스트만 생성한다. 도구가 있어야 읽고 편집하고 실행할 수 있다. 각 결과가 루프로 피드백된다.

### Tools (행동 수단) — 5가지 카테고리

1. FILE — 읽기, 편집, 생성, 이름변경, 재구성
2. SEARCH — 패턴/정규식 검색, 코드베이스 탐색
3. EXECUTION — 셸 명령, 서버 기동, 테스트, git
4. WEB — 웹 검색, 문서 가져오기, 에러 조회
5. CODE INTEL — 타입 오류, 정의 이동, 참조 (플러그인)

내장 도구 5가지 + 오케스트레이션 도구로 구성. 외부 도구는 MCP로 추가한다.

참고: claude-code-architecture-2-inputs.md, claude-code-architecture-3-execution.md
