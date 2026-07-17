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
  - `glossary/*/published/` — 지호님이 직접 블로그에 발행 완료한 glossary 파일. 이 위치는 "발행 완료" 상태 표시일 뿐, 내용 수정 금지를 뜻하지 않는다. 파일을 이 폴더로 옮기는 것(발행 여부 관리)은 지호님만 한다. 내용 수정(사실 보강, 오류 수정, 최신화 등)은 일반 glossary 파일과 동일하게 자유롭게 하되, 수정 후에는 변경 사항을 요약해서 알린다 — 이미 발행되어 블로그에 나가 있는 내용이라 조용히 바뀌면 안 되기 때문이다.
- `spring/` — Spring 학습 노트
- `java/` — Java 학습 노트
- `book/` — 기술 서적 독서 노트

## 문서 생성 규칙

### 트레이닝 문서는 타이트하게, 블로그는 프리하게
- `blog/` — 자유롭게 작성, 발행 강제하지 않음
- `jvm-training/`, `db-internals/` — 문서 생성 직후 Claude가 Q1(기본) 질문 1개를 즉시 던진다
- 트레이닝 문서는 이해도 테스트 3단계를 통과해야 Phase 완료로 인정한다
- 지호님이 답하지 않으면 문서는 생성됐지만 학습은 시작도 안 된 것으로 간주한다

### glossary 문서의 예시 사용 기준
- 다른 시스템(MySQL, Kafka, JVM 등)을 언급할 때는 그 시스템을 설명하려는 게 아니라, 현재 개념을 설명하기 위해 사용하는 것임을 확인한다
- "이런 것들이 있어요" 나열은 쓰지 않는다 — 나열 자체가 목적이면 쓰지 말 것
- 예시가 개념보다 많아지면 잘못된 것이다. 예시는 개념을 뒷받침하는 최소한으로만

### 용어집(glossary/) 관리 규칙
- 트레이닝/블로그 중 "이게 뭔지 모르겠다" 싶은 개념이 나오면 `glossary/`에 단독 파일로 정리한다
- 파일명은 개념명 소문자 영어로 (예: `buffer.md`, `transaction.md`)
- 트레이닝 문서에서 해당 개념이 등장하면 glossary 파일명을 텍스트로 언급한다 (링크 금지)
- 블로그 주제로 키우기엔 작지만, 그냥 흘려보내기엔 중요한 개념들을 여기에 쌓는다

### 기술 서적 챕터별 문서 구조

기술 서적을 챕터 단위로 학습할 때 두 종류의 문서를 만든다.

1. **키워드 파일** — 개념 하나당 파일 하나. 깊이 있는 개념 정의와 맥락. `glossary/books/책이름/키워드.md`
2. **챕터 종합 문서** — 챕터에서 배운 내용을 하나의 흐름으로 정리한 복습 문서. `chXX-챕터제목.md`

챕터 종합 문서 상단에는 이 챕터에서 생성된 키워드 파일 목록을 텍스트로 나열한다 (링크 금지).
로드맵 파일에는 챕터 완료 후 실제 생성된 파일 목록만 추가한다. 예상 키워드는 쓰지 않는다.

### 기술 서적 이미지 자리 표시 규칙

책에 다이어그램이나 Figure가 있고 문서 내용과 직접 연결되는 경우, 해당 위치에 자리 표시를 남긴다.
- 형식: `> 📷 Figure X-X (책 p.XX) — 설명`
- 책 페이지 번호는 PDF 페이지가 아니라 책 본문 페이지 번호 기준으로 적는다.
- 이미지는 지호님이 직접 책에서 캡처해서 넣는다. Claude가 대신 넣지 않는다.
- 문서(glossary, 종합 문서)에만 표시한다. 블로그 발행 시 실제 이미지로 대체한다.
- 책을 읽으면서 Figure가 있는 페이지를 발견하면 반드시 자리 표시를 삽입한다.

### 문서 간 참조 규칙
- 마크다운 링크(`[텍스트](경로)`) 사용 금지
- 이유: 파일 이동/이름 변경 시 링크가 깨지고 관리 포인트만 늘어난다
- 대신 `참고: redo-log.md` 처럼 파일명을 텍스트로만 언급한다

### 표 사용 금지
- 이 레포에서 새로 만들거나 수정하는 문서에는 마크다운 표를 쓰지 않는다
- 이유: 표는 블로그에 타이핑하기 어렵고, 문장이 아니라서 타이핑하며 생각을 정리하는 과정이 안 된다
- 대신 목록이나 문장으로 풀어쓴다
- 예외: 책이나 외부 자료에서 그대로 가져온 표는 원문 그대로 유지한다

### 문장 길이 제약
- 이 레포에서 새로 만들거나 수정하는 문서는 한 문장에 개념(사실/원인/결과) 하나만 담는다
- 절 개수를 기계적으로 세지 않는다. 서로 다른 개념이 한 문장에 얽혀 있으면 나누고, 인과관계로 자연스럽게 이어지는 한 흐름이면 굳이 끊지 않는다
- 문장을 나눌 때도 문장 간 인과·대조 관계(그래서/다만/즉)가 드러나게 쓴다. 단문을 단순 나열만 하면 오히려 관계가 안 보여서 더 읽기 어려워진다
- 이유: 한 문장에 여러 개념이 얽히면 읽으면서 앞부분을 잊어버린다. 특히 glossary는 개념 하나를 정확히 새기는 게 목적이라 문장이 길어지면 그 목적에 반한다
- 예외: 코드/커맨드/mermaid 블록 안의 텍스트, 인용/발췌 원문은 대상이 아니다

### mermaid 다이어그램 권장
- 아키텍처, 시퀀스, 데이터 흐름, 노드 간 관계처럼 구조나 흐름을 다루는 내용이 나오면 mermaid 다이어그램 사용을 적극 권장한다
- 적용 범위: `blog/`, `jvm-training/`, `db-internals/`, `glossary/` 전체
- 텍스트만으로는 관계가 잘 안 그려지는 설명(여러 컴포넌트 간 연결, 순서가 있는 상호작용, 트리/그래프 구조)일수록 mermaid로 대체할지 먼저 검토한다
- 이유: 텍스트 설명보다 구조를 훨씬 빠르게 파악할 수 있다 (k8s-nfs-storage.md에서 확인)
- 강제는 아니다 — 다이어그램으로 그릴 만큼 구조적이지 않은 내용까지 억지로 그리지 않는다

---

## 커밋/푸시 규칙

### 절대 자동 커밋/푸시 금지
- 수정 완료 후 변경 사항 요약 제시
- 사용자 명시적 승인 후에만 커밋/푸시

### git diff 자동 실행 금지
- 사용자가 요청할 때만 실행

---

## 블로그 작성 규칙

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