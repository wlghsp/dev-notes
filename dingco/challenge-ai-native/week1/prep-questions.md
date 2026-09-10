# 1주차 준비: 알아야 할 것들

week1-execution-guide.md로 실제 작업을 하기 전에 먼저 답할 수 있어야 하는 질문들. 대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18 (Next.js 14 블로그). 코드를 근거로 채운다.

---

## 1. claude.md 헌법

- claude.md는 왜 README와 별도로 존재하는가? 이 저장소에서 claude.md에 반드시 있어야 하지만 README에는 없어도 되는 정보는 무엇인가?

- CLAUDE.md는 AI를 위한 문서이고, README.md는 사람을 위한 문서
- claude.md에 반드시 있어야 하지만 README에는 없어도 되는 정보는 무엇인가?
  - 종합해서 AI 전용 컨텍스트 정보입니다.
  - AI를 위한 코딩 컨벤션 및 스타일 가이드, 엄격한 금지사항 및 안티 패턴, 프로젝트 아키텍처 및 디렉토리 규칙


- 이 저장소의 실제 실행 명령은 무엇인가? (`package.json`의 scripts를 근거로 dev/typecheck/build/test/e2e 각각 채우기)

: npm run + `dev/typecheck/build/test/e2e`

- "금지 변경"에 들어가야 할 경로는 무엇인가? (`missions/`, `challenge.json`, `.github/workflows/`가 왜 수정 불가인지 README/challenge.json 근거로 설명)

-  `missions/` : 미션 정의 자체이므로, 참가자가 문제를 자기한테 유리하게 바꾸면 챌린지가 무의미해짐 
- `challenge.json` : 챌린지 메터데이터/채점 설정이라 변조 시 부정행위로 간주될 수 있음
- `.github/workflows/` : CI에서 `npm test(=selfCheck)`를 실행하는 파이프라인의므로, 여길 고치면 자동 검사 자체를 우회하거나 무력화할 수 있음


- "완료 조건"은 무엇을 기준으로 적어야 하는가? (자동 검사가 `npm test`라는 것과 무슨 관계가 있는가)

: selfCheck에 지정된 명령의 통과 여부는 단일하고 검증 가능한 기준으로 적어야 합니다.

## 2. 스킬 설계

- `.claude/skills/<이름>/SKILL.md`의 프론트매터(`name`, `description`)는 왜 필요한가? Claude가 이 스킬을 언제 골라 쓰는지와 무슨 관계가 있는가?

: 점진적 공개 구조. 스킬을 골라쓰기 위함. 한번에 로드하면 컨텍스트가 차고 비용 낭비

- 이 저장소에서 "반복 작업"이라 부를 만한 것은 무엇인가? (`app/blog/posts/hello-ai-native.md`, `context-is-everything.md`의 프론트매터 구조를 보고, 새 글을 만들 때 매번 반복되는 부분이 무엇인지)

: 새 블로그 글 만들기

- 스킬이 만든 산출물이 `app/blog/posts.ts`의 `getPosts()`에 바로 잡히려면 무엇을 지켜야 하는가? (프론트매터 파싱 정규식, `publishedAt` 정렬 기준)

- 파일: app/blog/posts/<slug>.md, 확장자 .md
프론트매터: 첫 줄부터 --- ~ ---로 감싸고, key: value 형식으로 title, publishedAt, summary 포함
publishedAt은 YYYY-MM-DD 형식 필수 (문자열 비교로 정렬되기 때문)

## 3. Hook — PreToolUse

- PreToolUse Hook은 언제 실행되는가? PostToolUse와 실행 시점이 어떻게 다른가?

: PreToolUse Hook은 도구를 실제로 실행하기 직전에 실행됨
: PostToolUse 은 도구가 실행 완료된 후에 실행

- 이 저장소에서 "막아야 하는 변경"은 구체적으로 무엇인가? (missions/, challenge.json, .github/workflows/ 외에 더 있는가?)

: missions/, challenge.json, .github/workflows/ 세 가지는 §1과 동일 — 채점/검증 로직 자체이므로 참가자가 손대면 공정성이 깨짐.
추가로 막아야 할 후보:
- evidence/ 안의 기존(과거 제출) 파일 — 이미 제출한 증빙을 나중에 고쳐서 검증 결과를 조작하는 것 방지
- .claude/settings.json, Hook 스크립트 자체 — Hook이 스스로를 무력화하도록 자기 자신을 수정하는 것 방지
- package.json의 scripts.test — selfCheck가 실제로 실행하는 명령이므로, 여길 바꾸면 npm test 통과가 무의미해짐
(정확한 목록은 README/challenge.json을 다시 확인해 근거를 붙여야 함 — 위는 가설)

- Hook이 변경을 차단했다는 걸 exit code로 어떻게 표현하는가? (강의 7강 exit code 규약 — 공식 문서: https://code.claude.com/docs/en/hooks#exit-code-output)

: Exit code 2

- "일부러 위반하는 요청을 넣어 차단되는 출력을 남긴다"는 게 구체적으로 무슨 절차인가? (무엇을 시도하고, 무엇을 캡처해야 하는가)

금지 파일(예: challenge.json) 수정을 막는 PreToolUse Hook을 먼저 설치
Claude에게 그 파일을 고쳐달라고 일부러 요청
Hook이 exit code 2로 차단하는 순간의 stderr 메시지 + 거부 화면을 로그/스크린샷으로 캡처해서 저장

## 4. Hook — PostToolUse

- PostToolUse Hook으로 "파일 수정 직후 자동 실행"할 만한 작업은 이 저장소에 무엇이 있는가? (`npm run typecheck`? lint가 따로 있는가 확인)

: npm run typecheck

- PostToolUse가 실패하면(예: 타입 에러) 무엇이 어떻게 사용자에게 보여야 하는가?

: PostToolUse는 이미 실행된 도구 결과를 막을 수는 없으니, 실패 시 역할은 Claude에게 에러를 되돌려줘서 스스로 고치게 만드는 것입니다.

## 5. 제출 형식

- 이번 주차 evidence 파일명은 무엇인가? (`evidence/<브랜치와 같은 이름>.md`)

: evidence/submit/week-01__weekly-pr.md

- evidence/README.md의 각 섹션(변경/검증/선택 근거/근거형 질문/리뷰 반영)에 1주차 기준으로 각각 뭘 채워야 하는가?

1주차 미션(CLAUDE.md 채우기 + 반복 작업 스킬 + Hook) 기준으로 각 섹션에 채울 내용:

## 변경

CLAUDE.md에 프로젝트 구조/실행 명령/코딩 규칙/금지 변경/완료 조건을 채운 것
새 글 생성을 자동화하는 .claude/skills/(또는 유사) 스킬을 만든 것
금지 파일 수정을 막는 PreToolUse Hook, .ts/.tsx 수정 후 typecheck를 도는 PostToolUse Hook을 추가한 것
각각 "무엇을 왜" 했는지(예: 반복 작업을 스킬로 뽑은 이유, Hook을 건 이유) 서술
## 검증

npm test(tsc --noEmit && next build) 실행 결과
스킬로 새 글을 실제로 만들어보고 getPosts()에 잡히는지 확인한 절차/결과 (개발 서버에서 목록에 보이는지 등)
Hook 차단 테스트: 금지 파일(challenge.json 등) 수정 시도 → exit code 2로 차단된 로그/캡처
typecheck 실패를 일부러 만들어 PostToolUse가 에러를 잡아내는지 확인한 결과
## 선택 근거

스킬/Hook을 어떤 방식으로 구현했는지, 다른 방법(예: 수동 템플릿 복사, CI에서만 검사) 대신 이걸 고른 이유
아직 확인 못한 한계 (예: Hook이 특정 엣지 케이스는 못 잡는다든지)
## 근거형 질문

지금까지 대화에서 나온 질문들 — 반복 작업 식별, getPosts() 파싱 규칙, PreToolUse vs PostToolUse 실행 시점, 차단 exit code 규약, 위반 요청 재현 절차, typecheck 자동화 대상, 실패 시 사용자에게 보이는 방식 — 각각에 대해 최초 판단 → 연결한 코드/로그 → 검증 후 답변을 적음
## 리뷰 반영

첫 제출이면 그대로 자동 리뷰 수신 전

- 근거형 질문 3개(claude.md에 필요한 정보, PreToolUse/PostToolUse 구분 기준, Hook이 못 막은 실수)에 어떤 근거를 연결할 수 있는가?

: 각 질문에 연결할 근거 형태(실제 코드/로그는 작업 진행 후 채움):
1. claude.md에 필요한 정보 — package.json의 scripts 목록(dev/typecheck/build/test/e2e 실제 값), README/challenge.json에서 확인한 금지 경로 목록을 근거로 CLAUDE.md 작성본 자체를 근거로 제시
2. PreToolUse/PostToolUse 구분 기준 — 공식 문서(hooks#exit-code-output)의 실행 시점 설명 + 실제로 설치한 두 Hook 스크립트(파일 경로)와 각각이 언제 발동했는지 보여주는 로그
3. Hook이 못 막은 실수 — 실제로 Hook을 우회하거나 놓친 사례가 있었다면 그 로그/재현 절차. 아직 없다면 "아직 확인 못한 한계"로 선택 근거 섹션에 남긴 내용과 연결
