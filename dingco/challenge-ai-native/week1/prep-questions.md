# 1주차 준비: 알아야 할 것들

week1-execution-guide.md로 실제 작업을 하기 전에 먼저 답할 수 있어야 하는 질문들. 대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18 (Next.js 14 블로그). 코드를 근거로 채운다.

---

## 1. claude.md 헌법

- claude.md는 왜 README와 별도로 존재하는가? 이 저장소에서 claude.md에 반드시 있어야 하지만 README에는 없어도 되는 정보는 무엇인가?

:

- 이 저장소의 실제 실행 명령은 무엇인가? (`package.json`의 scripts를 근거로 dev/typecheck/build/test/e2e 각각 채우기)

:

- "금지 변경"에 들어가야 할 경로는 무엇인가? (`missions/`, `challenge.json`, `.github/workflows/`가 왜 수정 불가인지 README/challenge.json 근거로 설명)

:

- "완료 조건"은 무엇을 기준으로 적어야 하는가? (자동 검사가 `npm test`라는 것과 무슨 관계가 있는가)

:

## 2. 스킬 설계

- `.claude/skills/<이름>/SKILL.md`의 프론트매터(`name`, `description`)는 왜 필요한가? Claude가 이 스킬을 언제 골라 쓰는지와 무슨 관계가 있는가?

:

- 이 저장소에서 "반복 작업"이라 부를 만한 것은 무엇인가? (`app/blog/posts/hello-ai-native.md`, `context-is-everything.md`의 프론트매터 구조를 보고, 새 글을 만들 때 매번 반복되는 부분이 무엇인지)

:

- 스킬이 만든 산출물이 `app/blog/posts.ts`의 `getPosts()`에 바로 잡히려면 무엇을 지켜야 하는가? (프론트매터 파싱 정규식, `publishedAt` 정렬 기준)

:

## 3. Hook — PreToolUse

- PreToolUse Hook은 언제 실행되는가? PostToolUse와 실행 시점이 어떻게 다른가?

:

- 이 저장소에서 "막아야 하는 변경"은 구체적으로 무엇인가? (missions/, challenge.json, .github/workflows/ 외에 더 있는가?)

:

- Hook이 변경을 차단했다는 걸 exit code로 어떻게 표현하는가? (강의 7강 exit code 규약 — 공식 문서: https://code.claude.com/docs/en/hooks#exit-code-output)

:

- "일부러 위반하는 요청을 넣어 차단되는 출력을 남긴다"는 게 구체적으로 무슨 절차인가? (무엇을 시도하고, 무엇을 캡처해야 하는가)

:

## 4. Hook — PostToolUse

- PostToolUse Hook으로 "파일 수정 직후 자동 실행"할 만한 작업은 이 저장소에 무엇이 있는가? (`npm run typecheck`? lint가 따로 있는가 확인)

:

- PostToolUse가 실패하면(예: 타입 에러) 무엇이 어떻게 사용자에게 보여야 하는가?

:

## 5. 제출 형식

- 이번 주차 evidence 파일명은 무엇인가? (`evidence/<브랜치와 같은 이름>.md`)

:

- evidence/README.md의 각 섹션(변경/검증/선택 근거/근거형 질문/리뷰 반영)에 1주차 기준으로 각각 뭘 채워야 하는가?

:

- 근거형 질문 3개(claude.md에 필요한 정보, PreToolUse/PostToolUse 구분 기준, Hook이 못 막은 실수)에 어떤 근거를 연결할 수 있는가?

:
