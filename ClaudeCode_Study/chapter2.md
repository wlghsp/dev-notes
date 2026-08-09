# PART 01


## Subagent란 무엇인가

- 자기만의 컨텍스트 윈도우를 가진 격리 인스턴스
- Subagent는 특정 작업을 처리하는 격리된 Claude 인스턴스
- 자체 컨텍스트 윈도, 맞춤 시스템 프롬프트, 별도 도구 권한으로 독립 작업 후 요약만 돌려줍니다. 

## 왜 위임하는가

컨텍스트 오염이 품질을 깎는다.

- 검색 결과, 로그, 파일 내용이 메인 대화를 채우면 모델의 주의가 분산되고 응답 품질이 떨어집니다. 
- 다시 참조하지 않을 출력이 특히 낭비입니다. 

## Subagent 초기 컨텍스트의 다섯 구성

1. System Prompt - 에이전트 자신의 프롬프트 + 작업 디렉토리 등 환경 정보
2. Task Message - 메인 Claude가 작성한 위임 지시문, 대화 이력은 오지 않음
3. CLAUDE.md + memory 메인이 로드한 메모리 계층 전체
4. Git Status
5. Preloaded Skills - skills 필드에 지정한 스킬의 본문 전체

## 위임의 5대 가치

1. Preserve Context - 탐색과 구현 출력을 메인 대화 밖에 격리
2. Enfoce Constraints - 도구 제한으로 읽기 전용 등 안전성 강제
3. Reuse Configs - 사용자 레벨 정의로 전 프로젝트 재사용
4. Specialize - 도메인 특화 프롬프트로 성공률 상승
5. Control Costs - haiku 같은 경량 모델로 라우팅해 비용 절감
+ 팀 자산화 - 프로젝트 정의를 버전 관리로 팀과 공유, 공동 개선.

## Build-in / Explore

읽기 전용 탐색 전문 에이전트

- 코드 베이스 검색과 분석에 최적화된 읽기 전용 에이전트
- Claude가 수정 없이 코드를 이해해야 할 때 자동으로 위임


## 실행 모델 / Foreground vs Background

- v2.1.198부터 백그라운드가 기본값

- 서브에이전트는 기본적으로 백그라운드에서 돕니다. 
- 결과가 즉시 필요할 때만 포그라운드로 실행되며, 권한 프롬프트는 메인 세션에 떠오릅니다. 

## 상태 모델 / 종료 후에도 살아있다

- 각 호출은 새 인스턴스로 시작하지만, 완료된 에이전트의 대화 기록은 파일로 남아 이어서 재개 가능
- Explore와 Plan만 일회성 


# PART 02

## 정의 파일 해부

- YAML frontmatter(name, description, tools, model) + 본문(시스템 프롬프트)로 구성된 마크다운 한 장
- 필수 필드는 name, description 두 가지뿐. 정체성은 오직 name 필드로 결정됨

## 스코프 5계층과 배치 규칙

1. Managed - 조직 관리 설정, 관리자 배포
2. --agents 플래그 - 세션 한정, 디스크 저장 없음
3. Project - .claude/agents/, 팀 공유의 중심
4. User - ~/.claude/agents/, 내 모든 프로젝트
5. Plugin - my-plugin:name으로 스코프 등록
- 동일 스코프 중복 name은 하나만 로드, 파일 워처가 수 초 내 변경 반영 (재시작 불필요)

## name과 description / 자동 위임을 좌우하는 필드

- Claude는 description을 읽고 위임 여부를 판단하므로, "언제 쓰는지"를 명확히 써야 자동 위임 정확도가 올라감
- 좋은 예: "Use proactively after writing code", "MUST BE USED when tests fail" 처럼 시점과 능동 문구를 명시
- 나쁜 예: "코드를 리뷰하는 에이전트"처럼 역할만 있고 시점이 없는 설명

## tools / model / permissionMode

- tools 생략 시 메인의 전체 도구(MCP 포함) 상속, 지정 시 나열한 도구만 허용
- disallowedTools로 특정 도구만 차단 가능, mcp__서버명으로 MCP 서버 단위 통제
- model은 alias(sonnet/opus/haiku/fable), 전체 ID, inherit(기본값) 중 선택
- permissionMode: default/acceptEdits/auto/dontAsk/plan 중 선택, 단 부모가 bypass/acceptEdits/auto면 그쪽이 우선

## skills / mcpServers / memory 필드

- skills: 나열한 스킬의 본문 전체가 시작 컨텍스트에 주입되어 도메인 지식을 미리 장착
- mcpServers: 이 에이전트 전용 MCP 서버를 인라인 정의 가능 (메인에 노출 안 됨)
- memory: project(권장)로 세션을 넘어 학습이 축적되는 영속 메모리 사용 가능

## 공식 예시 / code-reviewer vs debugger

- code-reviewer: tools에 Edit 없이 읽기 전용으로 설계, 발견만 보고
- debugger: Edit 포함, 최소 수정 원칙을 프롬프트에 명문화하고 증상이 아닌 근본 원인을 고침


# PART 03

## Agent 도구 / 위임의 프리미티브

- v2.1.63에서 기존 Task 도구가 Agent로 개명 (Task(...) 표기는 별칭으로 계속 동작)
- 메인 Claude가 Agent 도구를 호출 → 위임 지시문 전달 → 완료 시 요약 + agent ID 회수

## 명시 호출 3단계 사다리

1. 자연어 - "test-runner 서브에이전트로 고쳐줘" (Claude가 판단)
2. @멘션 - `@agent-code-reviewer` 로 특정 에이전트 지목 보장
3. --agent - `claude --agent code-reviewer` 로 세션 전체를 그 에이전트로 실행

## 패턴 3종 / 병렬 리서치, 고볼륨 격리, 체이닝

- 병렬 리서치: 독립된 조사를 동시에 여러 서브에이전트로 스폰, 조사 경로가 서로 의존하지 않아야 함
- 고볼륨 격리: 수천 줄 로그/테스트 출력을 자식에서 소화하고 실패/요약만 메인으로 회수
- 체이닝: 한 에이전트의 결과를 다음 에이전트 지시문에 반영해 순차 연결

## Resume과 SendMessage / 중첩 서브에이전트

- 완료된 에이전트도 agent ID로 SendMessage하면 이전 도구 호출과 추론을 가진 채 재개 가능 (Explore/Plan 제외)
- v2.1.172부터 서브에이전트도 자기 서브에이전트를 만들 수 있음 (중첩 깊이 5단계 상한)

## /fork / 대화 전체를 물려받는 서브에이전트

- 지금까지의 대화 이력, 도구, 모델을 그대로 상속받아 백그라운드로 진행
- Named Subagent와 차이: fork는 전체 컨텍스트+메인과 캐시 공유(저렴), Named는 지시문으로 새 출발(정형화된 역할에 적합)

## 선택 가이드 / Main vs Subagent vs /btw

- 잦은 왕복/맥락 공유 작업은 Main 대화, 대량 출력·도구 제약이 필요한 작업은 Subagent, 빠른 질문은 /btw


# PART 04~08 — 실전 패턴 5종

## Pattern 1 / Code Reviewer

- 커밋 전 셀프 리뷰 표준화: git diff 중심, Critical/Warning/Suggestion 3단계로 우선순위 출력, 읽기 전용
- memory: project로 리뷰할수록 팀의 반복 이슈·컨벤션이 쌓이는 학습형 리뷰어로 진화
- 호출 경로 3가지: 대화형(자동 위임+@멘션), 헤드리스(`claude -p` + pre-push 훅), GitHub Actions(PR마다 자동 리뷰)
- 내장 `/code-review` 명령과는 대체가 아닌 보완 관계 (즉석 점검 vs 팀 표준 상비군)

## Pattern 2 / Tester

- test-writer: 커버리지 측정 → 갭 선정 → 컨벤션대로 작성 → 실행 → 의도 보존하며 수정, 5단계 워크플로
- isolation: worktree로 격리 실행 + maxTurns 40으로 폭주 방지

## Pattern 3 / Security Scanner

- security-scanner: 읽기 전용 tools + PreToolUse 훅(ro-guard.sh)의 이중 잠금, permissionMode: dontAsk로 무인 운용
- 인젝션, 인증/인가, 민감 데이터, 입력 검증, 설정 결함, 의존성 취약점 등 6대 관점 탐지를 본문에 명시
- pre-commit 게이트로 통합해 Critical 발견 시 커밋 자체를 차단 가능
- 오탐이 반복되면 게이트가 무시당하므로, 확인된 오탐을 memory에 축적해 재검토 순환을 만드는 게 핵심

## Pattern 4 / Docs Writer

- docs-writer: skills 프리로드(docs-style-guide, api-doc-template)로 스타일을 시작부터 고정
- README/API 문서/ADR/Changelog 자동 생성, 명령은 항상 실제 실행 후 검증하고 기재
- CI(paths 필터)로 API 변경과 문서를 동기화해 문서 부패를 구조적으로 차단

## Pattern 5 / Migration Bot

- migration-bot: isolation(worktree) + maxTurns 상한 + 배치 검증 절차 + 리뷰 회수(PR only), 4겹 안전장치가 기본값
- 라이브러리 업그레이드/import 경로 변경/deprecated API 교체 등 대규모 일괄 변경에 사용, 기계 치환과 사람 판단이 필요한 건을 지시문에서 명확히 분리
- 비용 최적화: 탐색은 haiku 워커, 실제 변환은 sonnet 본대로 역할 분담
- 내장 `/batch` 오케스트레이터(자동 조사+병렬 스폰+PR)와는 용도 분업: 대규모 균질 변경은 /batch, 세밀한 규칙 제어가 필요하면 migration-bot 단독 운용


# PART 09 Recap & Labs

## Chapter 2 핵심 요약

- 개념: 격리 컨텍스트에서 일하고 요약만 돌려주는 세션 내 워커
- 정의: 마크다운 한 장, 필수는 name과 description, 스코프 5계층
- 필드: tools, model, memory, skills, hooks, isolation의 조합 설계
- 디스패치: 자연어, @멘션, --agent 사다리와 병렬·체이닝·resume
- 고급: fork의 전체 상속과 캐시 공유, 중첩 깊이 5, SendMessage
- 패턴 5종: 리뷰어·테스터·스캐너·문서·마이그레이션의 검증된 골격

## 실습 랩 4종

1. 첫 Subagent 만들기 (15분) - Claude에게 생성 위임, @멘션 호출, 정의 수정 후 무재시작 반영 확인
2. 도구 권한 격리 (15분) - 읽기 전용 에이전트의 거동 관찰, disallowedTools, 훅 검증(exit 2) 체험
3. 디스패치와 병렬 (20분) - 자동 위임 관찰, 3개 병렬 리서치, 백그라운드 패널 조작, 체이닝
4. 메모리와 Fork (25분) - memory: project 켜기, 학습 순환 2회 체험, /fork로 대화 전체 상속 체험

## 다음 챕터 예고

- Chapter 3 Admin Setup: 배포 전략(조직 단위 설치, managed settings), 자격증명(SSO, Bedrock IAM), 사내망(프록시, 도메인 허용), 거버넌스(정책 계층, 감사 로그, 비용 관측)

