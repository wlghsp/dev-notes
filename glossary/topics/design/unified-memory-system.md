# Unified Memory System — 기업용 공유 AI 메모리

출처: Anthropic이 개발 중인 기업용 공통 메모리 아키텍처 (2025~2026 공개)  
관련 파일: andrej_karpathy_claude_md_explained.md

---

## 왜 필요한가

A부서 에이전트와 B부서 에이전트가 같은 종류의 시행착오를 각자 반복한다.  
`flaky-tests.md` 에 어떤 테스트가 불안정한지 기록했는데, 다른 팀은 그걸 모르고 또 삽질한다.  
시행착오 비용을 두 부서가 중복해서 지불할 필요가 없다.

해결책: 에이전트들이 배운 것을 공통 메모리에 쌓고, 새 에이전트는 그걸 읽고 시작한다.

---

## 아키텍처: Memory + Dreaming

Anthropic의 unified memory system은 두 가지 레이어로 구성된다.

**Memory — 실시간 업데이트**

에이전트가 작업하면서 team-memory를 읽고 쓴다.  
team-memory는 주제별 마크다운 파일로 관리된다.

- `deploy.md` — 배포 관련 절차와 주의사항
- `flaky-tests.md` — 불안정한 테스트 목록과 원인
- `repo-layout.md` — 레포 구조와 규칙

Agent A session, Agent B session, Agent C session이 동시에 이 파일들을 참조한다.  
세션이 새로 뭔가를 배우면 해당 파일에 실시간으로 기록한다.

**Dreaming — 배치 업데이트**

세션이 끝난 후 주기적으로 실행되는 오프라인 처리 단계다.  
이름이 "Dreaming"인 이유: 사람이 자는 동안 뇌가 기억을 정리하고 강화하는 것과 같다.

처리 순서:
1. Session transcripts 수집 — 여러 세션의 대화 기록을 모은다
2. Verify — 기록된 내용이 사실인지 검증한다
3. Organize — 중복 제거, 분류, 구조화한다
4. Enrich — 맥락을 보강하고 관련 개념과 연결한다
5. team-memory 업데이트 — 정리된 내용을 공통 파일에 반영한다

---

## 개인 레벨에서 이미 작동하는 버전

Claude Code의 auto-memory가 이 구조의 개인 버전이다.

`~/.claude/projects/<project>/memory/` 디렉토리에  
`user_role.md`, `feedback_no_tables.md` 같은 파일들이 쌓인다.  
새 대화를 시작하면 Claude가 `MEMORY.md` 인덱스를 읽고 관련 메모리를 로드한다.

Anthropic이 만들고 있는 건 이걸 팀/기업 레벨로 확장하는 것이다.

---

## Harvey AI의 구현 사례

법률 AI Harvey가 이 패턴을 먼저 구현했고, 기존 대비 6배 벤치마크 상승을 확보했다.

구체적으로 한 것:
- firm admin이 메모리 on/off 제어 (거버넌스)
- matter details, 관련 선례, 작업 선호도, 승인된 best practice를 메모리에 저장
- 새 에이전트라도 회사 맥락 안에서 바로 잘 작동

참고: harvey.ai/blog/memory-in-harvey

---

## 사내 간이 구현 가능성

Anthropic 공식 출시 전에 비슷한 구조를 직접 만들 수 있다.

**Memory 파트 (지금 바로 가능)**  
공유 git repo에 team-memory 디렉토리를 만들고,  
에이전트들이 작업할 때 CLAUDE.md에 "team-memory를 먼저 읽고 작업 후 관련 파일을 업데이트하라"고 명시한다.

**Dreaming 파트 (파이프라인 필요)**  
세션 transcript 수집 → LLM으로 인사이트 추출 → team-memory 업데이트  
Claude API + 배치 스케줄러(cron 또는 GitHub Actions)로 구현 가능하다.

**필요한 것:**
- 에이전트 세션 로그 저장 권한
- 배치 처리용 Claude API 호출
- team-memory 파일 관리 권한 체계 (누가 어떤 파일을 쓸 수 있는가)
