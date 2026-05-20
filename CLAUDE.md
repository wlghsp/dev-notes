# dev-notes 작업 가이드

## 이 레포의 목적
블로그 초안, JVM 트레이닝, DB Internals 트레이닝, 개인 학습 노트 전용 레포.
klid-middlewares(업무 레포)와 분리되어 있음.

목표: CS 기초부터 내부 구조까지 깊게 이해하는 개발자. 미국 빅테크 수준의 기술 역량.

## 폴더 구조
- `blog/` — 블로그 초안 (발행 전 원고)
- `blog/published/` — 발행 완료된 글
- `jvm-training/` — JVM 트레이닝 시리즈 및 로드맵
- `db-internals/` — DB Internals 트레이닝 시리즈 및 로드맵
- `glossary/` — 용어집 (주제별 개념 정의, 트레이닝/블로그와 연결)
- `spring/` — Spring 학습 노트
- `java/` — Java 학습 노트
- `book/` — 기술 서적 독서 노트

## 문서 생성 규칙

### 트레이닝 문서는 타이트하게, 블로그는 프리하게
- `blog/` — 자유롭게 작성, 발행 강제하지 않음
- `jvm-training/`, `db-internals/` — 문서 생성 직후 Claude가 Q1(기본) 질문 1개를 즉시 던진다
- 트레이닝 문서는 이해도 테스트 3단계를 통과해야 Phase 완료로 인정한다
- 지호님이 답하지 않으면 문서는 생성됐지만 학습은 시작도 안 된 것으로 간주한다

### 표(Table) 사용 금지
- 모든 문서에서 마크다운 표를 사용하지 않는다 (glossary/, jvm-training/, db-internals/, blog/ 전부)
- 이유: 표는 한눈에 보기엔 좋지만, 직접 타이핑하기 어렵고 문장이 아니라 체화가 안 된다. 도움된 적 없음
- 대신 번호 목록, 들여쓰기, 문장형 설명, 코드 블록으로 표현한다

### glossary 문서의 예시 사용 기준
- 다른 시스템(MySQL, Kafka, JVM 등)을 언급할 때는 그 시스템을 설명하려는 게 아니라, 현재 개념을 설명하기 위해 사용하는 것임을 확인한다
- "이런 것들이 있어요" 나열은 쓰지 않는다 — 나열 자체가 목적이면 쓰지 말 것
- 예시가 개념보다 많아지면 잘못된 것이다. 예시는 개념을 뒷받침하는 최소한으로만

### 용어집(glossary/) 관리 규칙
- 트레이닝/블로그 중 "이게 뭔지 모르겠다" 싶은 개념이 나오면 `glossary/`에 단독 파일로 정리한다
- 파일명은 개념명 소문자 영어로 (예: `buffer.md`, `transaction.md`)
- 트레이닝 문서에서 해당 개념이 등장하면 glossary 파일명을 텍스트로 언급한다 (링크 금지)
- 블로그 주제로 키우기엔 작지만, 그냥 흘려보내기엔 중요한 개념들을 여기에 쌓는다

### 문서 간 참조 규칙
- 마크다운 링크(`[텍스트](경로)`) 사용 금지
- 이유: 파일 이동/이름 변경 시 링크가 깨지고 관리 포인트만 늘어난다
- 대신 `참고: redo-log.md` 처럼 파일명을 텍스트로만 언급한다

---

## 커밋/푸시 규칙

### 절대 자동 커밋/푸시 금지
- 수정 완료 후 변경 사항 요약 제시
- 사용자 명시적 승인 후에만 커밋/푸시

### git diff 자동 실행 금지
- 사용자가 요청할 때만 실행

---

## 블로그 작성 규칙

- 개념 설명 시 **Mermaid 다이어그램을 반드시 포함**한다
- 흐름도, 메모리 구조, 계층 관계 등 시각화가 가능한 것은 모두 Mermaid로 표현한다
- 텍스트만으로 설명하지 않는다 — 다이어그램 먼저, 설명은 보완

### 블로그 실습 캡쳐 유도 규칙
- 블로그 글에 SQL, 명령어, 코드 실행이 포함된 경우 — Claude가 글 완성 후 반드시 묻는다:
  "이 부분 직접 실행해보셨나요? 결과 캡쳐 있으면 넣으면 글의 신뢰도가 올라가요."
- 필수는 아니지만, 캡쳐가 있으면 `> 📷 **[실행 결과]**` 자리에 이미지로 대체하도록 안내한다
- 캡쳐 없이 발행해도 되지만, 있으면 반드시 넣도록 권장한다

---

## JVM / DB Internals 트레이닝 규칙

### 기본 원칙
- 트레이닝은 지호님이 준비됐을 때 한다. Claude가 먼저 압박하지 않는다.
- 지호님이 트레이닝 모드로 들어오면 그때 순서와 완료 기준을 안내한다.

### 트레이닝 모드일 때만 적용
- 순서: 문서 완성 → 블로그 발행 → 이해도 테스트
- 문서 완성 후 테스트를 먼저 던지지 않는다. 블로그 발행 완료 후 테스트 진행
- 이해도 테스트는 3단계 (Q1 기본 / Q2 심화 / Q3 실전)
- DB 트레이닝은 "MySQL에서 이렇게 쓴다"보다 "왜 이렇게 동작하는가"를 중심으로

### 중간 통합 테스트
- 여러 Phase가 완료되면 단일 Phase 테스트 외에 통합 테스트를 진행한다
- 예시: Phase 0~2 완료 후 → "INSERT 하나가 발생했을 때 Page, Buffer Pool, B-Tree가 어떻게 엮이는가 설명해봐"
- 통합 테스트는 지호님이 요청할 때 진행. Claude가 먼저 제안하지 않는다.

### 트레이닝 현황 파일
- JVM 로드맵: `jvm-training/TRAINING_ROADMAP.md`
- DB Internals 로드맵: `db-internals/TRAINING_ROADMAP.md`


# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.