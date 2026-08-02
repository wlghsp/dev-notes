출처: https://github.com/whchoi98/claude-code-workshop

위 레포의 pdf를 보면서 정리한 내용이니다.

# PART 01

---

## Claude Code 정의 

> Claude Code는 코드베이스를 읽고, 파일을 편집하고 명령을 실행하며 개발 도구와 통합되는 Anthropic의 Agentic Coding 도구

- 자연어 지시를 받아 멀티스텝 작업을 자율적으로 완수

---

## Anthropic

> Anthropic은 안전하고 신뢰 가능한 AI를 목표로 하는 프론티어 AI 기업

- Claude 모델 패밀리와 Claude Code, Agent SDK를 직접 개발

---

## Claude 모델 패미리

- Claude Fable 5 : 최상위 Mythos-class, 장시간 자율 세션 특화
- Claude Opus 4.8 : 깊은 추론, 복잡한 아키텍처 결정
- Claude Sonnet 5 : 기본 모델, 네이티브 1M 컨텍스트
- Claude Haiku 4.5 : 빠른 응답, 비용 효율

---

## 모델 Alias 시스템

> 버전 번호 대신 별칭으로 모델 선택

- default, best, fable / opus, sonnet / haiku 등

---

## Agentic AI 개념

- Agentic AI는 사용자의 목표를 받아 스스로 계획을 세우고, 도구를 사용하며, 결과를 관찰하면서 목표 달성까지 자율적으로 작동하는 AI 시스템

---

## Agentic Loop 3단계

> 작업을 주면 Claude는 세 단계를 섞어가며 수십 개의 행동을 연결
> 사용자는 언제든 Esc로 개입해 방향 조정 가능

- Step 1 - Gather Contxt
: 파일 검색과 읽기로 코드 이해, CLUADE.md와 메모리 로드
- Step 2 - Take Action
: 파일 편집, 명령 실행, 커밋 등 실제 변경 수행
- Step 3 - Verify Result
: 테스트 실행, 타입 검사, 결과 관찰 후 다음 판다
- LOOP - Repeat / Steer
: 완료까지 반복, Esc로 중단하거나 지시 추가 가능

---

## 실행 환경 스펙트럼

> 동일 엔진, 모든 곳에서 (Use Claude Code Everywhere)

- Terminal CLI : 모든 기능의 기준점. 파이프와 스크립트 등 Unix 조합성.
- 그외 : VS Code / Cursor, JetBrains, Desktop App, Web / iOS, Slack / CI / Chrome

---

## 왜 터미널 기반인가

CLI가 여전히 기준점인 이유

1. 도구 접근성: 빌드, git, 패키지 매니저 등 개발도구가 모두 CLI에 존재
2. 조합 가능성: 파이프, 리다이렉트, jq와 결합해 워크플로를 자유롭게 구성
3. 자동화 친화 : -p 헤드리스 모드로 CI/CD 와 스크립트에 즉시 통합.
4. IDE 중립 : 특정 에디터에 종속되지 않고 모든 개발 환경과 공존

---

## Claude Code의 4대 핵심 가치

1. 깊은 코드 이해 
: 단일 파일이 아닌 프로젝트 전체를 자율 탐색하며 컨텍스트 구축
2. 실제 작업 완수
: 제안에 그치지 않고 파일 수정, 테스트 실행, PR 생성까지 수행.
3. 안전한 권한 모델
: allow, ask, deny 규칙과 체크포인트로 동작 범위를 통제
4. 확장 가능한 통합
: Skills, Subagents, MCP, Hooks, SDK로 워크플로에 맞춰 확장

---

## 자율성 스펙트럼

Plan에서 Auto까지, 권한 모드 4종
> Shift + Tab으로 모드를 순환

1. Plan : 소스 수정 없이 탐색과 계획만, 승인 후 실행
2. Default : 파일 편집과 셀 명령 전 매번 질문
3. Accept Edits : 편집과 mkdir 등 파일 작업 자동, 그 외 질문
4. Auto : 백그라운드 안전 검사로 전 행동 평가, 리서치 프리뷰

---

## Claude Code가 가장 큰 가치를 발휘하는 시나리오 

1. 디버깅 : 에러 메시지로부터 원인을 추적하고 수정안을 제시, 적용.
2. 리팩토링 : 대규모 코드베이스의 일관된 패턴 변경과 구조 개선.
3. 신기능 개발: 요구사항으로부터 계획, 구현, 테스트까지 일괄 처리
4. 코드베이스 학습 : 낯선 프로젝트의 구조와 흐름을 자연어 대화로 학습
5. 테스트 작성 : 커버리지 갭 분석과 자동 테스트 생성, 실패 수정.
6. DevOps 자동화 : 쉘 스크립트, IaC, CI/CD 파이프라인 작성과 점검

---

## 데이터 보안 및 프라이버시

> 로컬 실행 시 코드는 사용자의 머신에 있고, 모델 추론에 필요한 컨텍스트만 API로 전송. 전송 구간과 저장 구간 모두 암호화

--

# PART 02

---

## 전체 아키텍처 / Agentic Harness

> 언어 모델을 코딩 에이전트로 만드는 4계층 

1. Interface : Terminal CLI / VS Code / JetBrains / Desktop / Web / Slack / CI, 동일엔진의표면
2. Harness : 도구실행, 권한검사, 컨텍스트와 메모리관리, 체크포인트, 세션 저장
3. Model : Fable 5, Opus 4.8, Sonnet 5, Haiku 4.5, alias로선택하고/model로전환
4. Provider : Claude API / Amazon Bedrock / Google Vertex AI / Microsoft Foundry / Gateway

---

## 실행환경 3종

1. Local : 기본값. 내 머신에서 실행, 파일과 도구, 환경에 완전 접근
2. Cloud: Anthropic 관리VM에서실행. 작업오프로드, 원격저장소작업.
3. Remote Control : 실행은내머신, 조작은브라우저. 로컬 유지 + 웹UI.

---

## Agent SDK 의 위치

> Agent SDK는Claude Code의하네스를 Python과 TypeScript 라이브러리로 노출.
> CLI가 쓰는 것과 동일한 도구와 권한 체계 를 앱에 내장 가능

---

## API 통신 경로

> 하네스는 설정된 공급자 하나로 추론 요청을 보냅니다.
> 환경 변수 스위치로 경로를 선택하고, 조직은 게이트웨이로 중앙 집중할 수 있음 

---

## Tool System

다섯 가지 도구 카테고리

> 도구가 없으면 모델은 텍스트만 반환
> 도구가 있어야 읽고, 고치고, 실행하고, 검색하며 루프를 돌 수 있음 

1. File Ops : Read, Edit, Write, NotebookEdit
- Read, Edit, Write : 파일 읽기, 정밀 수정, 신규 생성

2. Search : Glob, Grep, 코드베이스 탐색
- Glob/Grep : 패턴 파일 찾기, 내용 정규식 검색
3. Execution : Bash, PowerShell, git, 테스트
- Bash/PowerShell : 쉘 명령 실행, Windows 네이티브 지원
4. Web : WebSearch, WebFetch, 문서 조회
- WebSearch/WebFetch : 웹 검색과 URL 콘텐츠 조회
5. Code Intel : LSP:타입오류, 정의 이동
- Agent / SendMessage : 서브에이전트생성, 에이전트간메시지 
- LSP / Monitor / Cron : 코드 인텔리전스, 로그 감시, 예약

### 2026년 상반기 추가된 능력

1. LSP : 편집 직후 타입 오류 확인, 정의 이동, 참조 찾기
2. Monitor : 백그라운드 명령 출력을 실시간 관찰하고 반응
3. AskUserQuestion : 요구사항이 모호할 때 객관식으로 사용자에게 질문
4. Aritifact : 세션 결과를 조직 내 공유 가능한 라이브 페이지로 발행


---

## MCP Tool Integration

외부 서비스를 도구로 연결

> MCP는 AI 도구를 외부 데이터 소스에 연결하는 개방형 표준
> GitHub, Slack, 사내 시스템의 기능이 Claude의 도구로 등록됨

1. CONNCET - 서버 연결
: claude mcp add로 stdio, HTTP 서버를 등록하고 /mcp로 상태 확인.
2. SCALE - Tool Search
: 도구 정의는 기본 지연 로딩, 사용 시점에 온디맨드 로드로 컨텍스트 절약
3. AUTH - 인증
: claude mcp login으로 셸에서 OAuth 인증, 조직은 허용 목록 통제.
4. GOVERN - 관리
: managed 설정으로 서버 allowlist, denylist를 조직 차원에서 강제

---

## Context Window 구성

```shell
System instructions : 하네스 기본 지침
CLAUDE.md (계층 병합) : 프로젝트, 사용자, 조직 지침
Auto memory(MEMORY.md) : 첫 200줄 또는 25KB
Skills 설명부 : 본문은 사용 시점 로드
MCP 도구 이름 : 정의는 온디맨드 로드
대화 이력 + 도구 출력 : 세션이 길수록 증가
```

## Context Compaction 상세

한계에 다가갈 때의 자동 처리

> 한계에 접근하면 오래된 도구 출력부터 정리하고, 필요하면 대화를 요약
> 초반의 세부 지시는 사라질 수 있으니 영속 규칙은 CLAUDE.md에 둠

- STAGE 1 - 임계 감지 : 윈도우 사용량이 한계에 접근
- STAGE 2 - 출력 정리 : 오래된 도구 출력부터 제거
- STAGE 3 - 대화 요약 : 요청과 핵심 코드는 보존
- GUARD - Thrashing 방지 : 반복 실패 시 루프 대신 오류 표시

---

## Checkpointing

모든 파일 편집은 되돌릴 수 있다

> Claude가 파일을 편집하기 전에 해당 파일의 현재 내용을 스냅샷으로 남김
> Esc 두 번이나 /rewind 로 이전 상태로 되돌림 

- STORAGE - JSONL 세션 기록
: ~/.cluade/projects/ 아래 평문 기록으로 resume와 fork 지원.

---

## Security Architecture

네 겹의 방어선

1. Layer 1 - Permissions : allow, ask, deny 규칙과 4가지 모드로 행동 사전 통제.
2. Layer 2 - Sandboxing : 샌드박스 Bash로 파일 시스템과 네트워크 격리 실행.
3. Layer 3 - Checkpoints : 편집 전 스냅샷으로 사후 복구 경로 확보.
4. Layer 4 - Audit : 세션 기록, Hooks, OTel로 행위 추적과 정책 검증.

---

## Logging & Telemetry

로컬 기록과 OpenTelemetry

> 세션은 로컬 JSONL로 기록되고, 조직 모니터링은 OpenTelemetry 표준으로 내보냅니다.

- LOCAL RECORDS
: ~/.claude/projects/ 세션 JSONL. resume, fork, rewind의 기반. claude --debug로 상세 로그 확인. /export로 대화 내보내기 가능.

- ORG TELEMETRY (OTel)
: 토큰, 비용, 도구 사용 메트릭을 OTLP로 CloudWatch, Datadog 등에 연계. 사용자·팀 단위 대시보드 구성. 설정 방법은 챕터 3에서 실습.

---

# PART 05

## 첫 프롬프트 작성 요령

공식 Best Practices 4원칙

> DELEGATE, DON'T DICTATE
> 유능한 동료에게 위임하듯 맥락과 방향을 주고 세부 경로는 맡깁니다. 어떤 파일을 읽을지까지 지시할 필요는 없습니다.

- TIP 1 - 구체적으로
: 관련 경로, 제약, 참고 패턴을 처음부터 명시해 교정 횟수 절감.
- TIP 2 - 검증 기준 제공
: 테스트 케이스, 기대 출력 등 스스로 확인할 기준 제공.
- TIP 3 - 탐색과 분리
: 복잡한 문제는 Plan 모드로 조사 먼저, 코딩은 그다음.
- TIP 4 - 대화로 교정
: 완벽한 첫 프롬프트보다 중간 개입과 반복 교정이 빠름.

---

## 명확한 vs 모호한 지시

같은 목표, 다른 결과

> SPECIFICITY WINS
> 범위와 완료 기준이 담긴 지시가 1회 성공률을 결정합니다.

- 모호한 지시 (교정 비용 큼)
: 로그인 버그 고쳐줘 / 테스트 좀 추가해줘 / 코드 정리해줘. 탐색 비용과 방향 이탈 위험 증가.
- 명확한 지시 (1회 성공률 상승)
: 만료 카드 결제 실패, src/payments/ 토큰 갱신부터 조사 / validateEmail 구현, 3개 케이스 통과 후 테스트 실행 / src/legacy만, 동작 변경 없이 이름 규칙 통일. 범위와 완료 기준이 함께 전달됨.

---

# PART 8

---

# PART 9


