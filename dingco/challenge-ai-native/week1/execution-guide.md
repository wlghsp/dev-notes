# 1주차 실행 가이드

prep-questions.md 답변을 마친 뒤 실제로 진행하는 순서. 대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18, 브랜치는 이미 만들어진 `submit/week-01__weekly-pr`를 이어서 쓴다. 아래 코드 블록은 그대로 파일에 붙여넣고 저장소 상황에 맞게 다듬는 용도다.

---

## 0단계: 시작 전 확인

```bash
git branch --show-current   # submit/week-01__weekly-pr 이어야 함
npm ci                      # node_modules가 비어 있으면 typecheck/test가 전부 "Cannot find module" 에러로 실패한다
find .claude -type f        # 지금은 README.md만 있어야 정상
cat claude.md                # 헌법 항목 목록만 있고 비어 있어야 정상
```

## 1단계: claude.md 작성

저장소 루트의 `claude.md`(소문자)를 아래 내용으로 채운다. 루트의 `CLAUDE.md`(대문자)와는 다른 파일이다.

```markdown
# 프로젝트 헌법

## 프로젝트 구조

- `app/` — Next.js 14 App Router 페이지와 컴포넌트
- `app/blog/posts/*.md` — 글 원본. 프론트매터에 `title`, `publishedAt`(YYYY-MM-DD), `summary` 필수
- `app/blog/posts.ts` — 글 목록을 읽어 파싱하는 로더 (`getPosts()`, `getPost(slug)`)
- `app/blog/page.tsx`, `app/blog/[slug]/page.tsx` — 글 목록/상세 페이지
- `tests/` — Playwright E2E 시나리오
- `.claude/skills/` — 반복 작업 스킬
- `.claude/settings.json`, `.claude/hooks/` — Hook 설정과 스크립트

## 실행 명령

- 개발 서버: `npm run dev`
- 타입 검사: `npm run typecheck`
- 빌드: `npm run build`
- 전체 검사(자동 검사와 동일): `npm test` (`tsc --noEmit && next build`)
- E2E: `npm run e2e` (최초 1회 `npx playwright install chromium`)

## 코딩 규칙

- 새 글은 `app/blog/posts/<slug>.md`에 추가하고, frontmatter는 `---`로 감싸 `title`/`publishedAt`/`summary` 3개를 채운다
- `publishedAt`은 반드시 `YYYY-MM-DD` 형식으로 쓴다. `posts.ts`가 문자열 비교로 정렬하므로 형식이 어긋나면 목록 순서가 깨진다
- 패키지 매니저는 npm으로 고정한다 (자동 검사가 `npm ci && npm test`로 돈다)

## 금지 변경

- `missions/` — 미션 문제 정의 자체. 참가자가 문제를 바꾸면 챌린지가 무의미해진다
- `challenge.json` — 채점 메타데이터. 수정하면 부정행위로 간주된다
- `.github/workflows/` — 자동 검사 파이프라인. 수정하면 검사 자체를 무력화할 수 있다

## 완료 조건

- `npm test`가 통과한다
- 이번 주차 요구사항(claude.md 작성, 스킬 1개, PreToolUse/PostToolUse Hook)이 실제로 동작하는 산출물로 남아 있다
```

작성 후 **적은 명령을 전부 직접 실행**해서 통과 여부를 확인하고, 실행 로그를 남겨둔다.

```bash
npm run typecheck
npm test
```

## 2단계: 스킬 만들기 — 새 블로그 글 생성

```bash
mkdir -p .claude/skills/create-post
```

`.claude/skills/create-post/SKILL.md`:

```markdown
---
name: create-post
description: app/blog/posts 아래에 title/publishedAt/summary 프론트매터를 갖춘 새 블로그 글을 만든다. "새 글 써줘", "블로그 포스트 추가" 같은 요청에 사용한다.
---

# 새 블로그 글 생성

## 언제 쓰는지

`app/blog/posts/` 아래에 새 마크다운 글을 추가해야 할 때.

## 절차

1. 글 제목을 바탕으로 slug를 만든다 (소문자, 하이픈 구분, 영문 권장)
2. `app/blog/posts/<slug>.md` 파일을 아래 형식으로 만든다:

   ```markdown
   ---
   title: <글 제목>
   publishedAt: <YYYY-MM-DD>
   summary: <한 줄 요약>
   ---

   <본문>
   ```

3. `publishedAt`은 오늘 날짜 또는 사용자가 지정한 날짜를 `YYYY-MM-DD`로 쓴다 (문자열 정렬 기준이므로 형식을 반드시 지킨다)
4. `npm run dev`로 `/blog`에서 새 글이 목록에 올바른 순서로 보이는지 확인한다
```

스킬을 실제로 실행해 글 1개를 만들고, 결과를 확인한다.

```bash
npm run dev
# 브라우저에서 http://localhost:3000/blog 접속, 새 글이 목록/상세에 보이는지 확인 후 스크린샷 또는 콘솔 로그로 남김
```

## 3단계: PreToolUse Hook — 금지 파일 수정 차단

```bash
mkdir -p .claude/hooks
```

`.claude/hooks/block-protected-paths.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT="$(cat)"
FILE_PATH="$(echo "$INPUT" | node -e "
  let d='';
  process.stdin.on('data', c => d += c);
  process.stdin.on('end', () => {
    const j = JSON.parse(d);
    console.log(j.tool_input?.file_path ?? '');
  });
")"

if [[ -z "$FILE_PATH" ]]; then
  exit 0
fi

case "$FILE_PATH" in
  */missions/*|missions/*|*/challenge.json|challenge.json|*/.github/workflows/*|.github/workflows/*)
    echo "차단됨: $FILE_PATH 는 금지 변경 경로입니다 (missions/, challenge.json, .github/workflows/)." >&2
    exit 2
    ;;
  *)
    exit 0
    ;;
esac
```

```bash
chmod +x .claude/hooks/block-protected-paths.sh
```

**주의 — 실제로 겪은 패턴 매칭 버그**: `*/challenge.json`처럼 슬래시로 시작하는 패턴은 `file_path`가 **상대 경로(슬래시 없이 `challenge.json`만)**로 들어올 때 매칭되지 않는다. Claude Code가 프로젝트 루트 파일을 Edit할 때 절대 경로가 아니라 상대 경로로 `file_path`를 넘기는 경우가 있어, `*/challenge.json`만 있으면 Hook이 뚫린다. 반드시 `challenge.json`(슬래시 없는 패턴)도 같이 넣어야 한다. 아래처럼 직접 검증할 수 있다:

```bash
echo '{"tool_input":{"file_path":"challenge.json"}}' | .claude/hooks/block-protected-paths.sh; echo "exit: $?"
echo '{"tool_input":{"file_path":"'"$(pwd)"'/challenge.json"}}' | .claude/hooks/block-protected-paths.sh; echo "exit: $?"
```
두 경우 모두 `exit: 2`가 나와야 한다.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/block-protected-paths.sh"
          }
        ]
      }
    ]
  }
}
```

**일부러 위반하는 요청을 넣어 차단 로그를 남긴다.** Claude에게 `challenge.json`을 수정해달라고 요청하고, exit code 2로 막히는 순간의 stderr 메시지와 거부 화면을 캡처한다. 이 캡처가 없으면 "통과만 확인했다"는 지적을 받는다.

## 4단계: PostToolUse Hook — 수정 직후 typecheck 자동 실행

`.claude/hooks/typecheck-after-edit.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT="$(cat)"
FILE_PATH="$(echo "$INPUT" | node -e "
  let d='';
  process.stdin.on('data', c => d += c);
  process.stdin.on('end', () => {
    const j = JSON.parse(d);
    console.log(j.tool_input?.file_path ?? '');
  });
")"

case "$FILE_PATH" in
  *.ts|*.tsx)
    echo "PostToolUse: $FILE_PATH 수정 감지, typecheck 실행" >&2
    npm run typecheck
    ;;
  *)
    exit 0
    ;;
esac
```

```bash
chmod +x .claude/hooks/typecheck-after-edit.sh
```

`.claude/settings.json`의 `hooks`에 `PostToolUse` 추가:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/block-protected-paths.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/typecheck-after-edit.sh" }
        ]
      }
    ]
  }
}
```

두 가지 실행 로그를 남긴다:
- 정상 케이스: `.ts`/`.tsx` 파일 수정 후 Hook이 typecheck를 돌려 통과하는 로그
- 실패 케이스: 일부러 타입 에러를 내는 수정을 해서 Hook이 잡아내고, 그 에러가 Claude에게 다시 전달돼 스스로 고치는 흐름까지 로그

## 5단계 (선택 확장, 이번 제출에서는 스킵): dispatcher 패턴

이번 제출에서는 진행하지 않기로 함. missions/README.md에도 선택 확장으로 명시되어 있고 필수 제출 증거·통과 기준에는 포함되지 않는다. 아래는 나중에 여유가 생기면 참고할 설계 메모로 남겨둔다.

필수 항목(1~4단계)을 모두 마친 뒤 진행. 지금은 `block-protected-paths.sh`(PreToolUse)와 `typecheck-after-edit.sh`(PostToolUse)가 각자 파일 경로를 보고 분기하는 같은 패턴을 따로 갖고 있다. dispatcher 패턴은 이 "경로를 보고 분기한다"는 로직을 스크립트 하나로 모으고, 어느 이벤트(Pre/Post)에서 왔는지만 인자로 구분해 처리를 라우팅하는 방식이다.

`.claude/hooks/dispatcher.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

HOOK_EVENT="$1"   # settings.json이 "PreToolUse" 또는 "PostToolUse"로 넘겨준다
INPUT="$(cat)"
FILE_PATH="$(echo "$INPUT" | node -e "
  let d='';
  process.stdin.on('data', c => d += c);
  process.stdin.on('end', () => {
    const j = JSON.parse(d);
    console.log(j.tool_input?.file_path ?? '');
  });
")"

[[ -z "$FILE_PATH" ]] && exit 0

case "$FILE_PATH" in
  */missions/*|*/challenge.json|*/.github/workflows/*)
    if [[ "$HOOK_EVENT" == "PreToolUse" ]]; then
      echo "차단됨: $FILE_PATH 는 금지 변경 경로입니다." >&2
      exit 2
    fi
    ;;
  *.ts|*.tsx)
    if [[ "$HOOK_EVENT" == "PostToolUse" ]]; then
      echo "PostToolUse: $FILE_PATH 수정 감지, typecheck 실행" >&2
      npm run typecheck
    fi
    ;;
  *)
    exit 0
    ;;
esac
```

```bash
chmod +x .claude/hooks/dispatcher.sh
```

`.claude/settings.json` — 두 Hook이 같은 스크립트를 이벤트 이름만 다르게 인자로 넘기도록 교체:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/dispatcher.sh PreToolUse" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/dispatcher.sh PostToolUse" }
        ]
      }
    ]
  }
}
```

### 통합 후 반드시 재검증한다

dispatcher로 합치면서 기존 로직이 깨질 수 있으므로, 3·4단계에서 이미 확보한 증거를 **같은 방식으로 다시 재현**해서 통합 전후 결과가 같은지 확인한다.

- 금지 경로(예: `challenge.json`) 수정을 일부러 요청 → exit code 2로 차단되는지 (3단계와 동일한 테스트)
- `.tsx` 파일에 타입 에러를 일부러 넣고 → PostToolUse가 typecheck로 잡아내는지 (4단계와 동일한 테스트, tsconfig include 범위 안인 `app/` 아래에서 진행하고 확인 즉시 되돌린다)
- 두 재검증 결과가 기존과 같으면 "통합 후에도 동작이 유지된다"는 근거로 evidence에 남긴다

### 기존 스크립트 정리

`block-protected-paths.sh`, `typecheck-after-edit.sh`를 dispatcher.sh로 대체했다면 삭제한다. 남겨두면 dispatcher가 실제로 그 두 로직을 대체했다는 게 diff에서 불명확해지므로, 삭제하고 `.claude/settings.json`이 dispatcher.sh만 가리키는 상태로 정리하는 쪽을 권장한다.

## 6단계: evidence 파일 작성

`evidence/week-01__weekly-pr.md`를 evidence/README.md 틀에 맞춰 작성한다.

- `## 변경`: claude.md 작성 / create-post 스킬 생성 / PreToolUse·PostToolUse Hook 추가, 각각 "무엇을 왜" 했는지
- `## 검증`: `npm test` 결과, 스킬로 만든 글이 목록에 잡히는 절차/결과, Hook 차단 로그, typecheck 실패→감지 로그
- `## 선택 근거`: Hook을 셸 스크립트로 구현한 이유(예: 별도 언어 런타임 없이 바로 실행), 버린 대안, 아직 확인 못한 한계
- `## 근거형 질문`: 질문 1~3에 최초 판단 → 연결한 코드/로그 → 검증 후 답변 순서로 (하단 메모 참고)
- `## 리뷰 반영`: 최초 제출은 "자동 리뷰 수신 전". **PR에 리뷰/self-check 결과가 달리면 그 즉시 이 섹션을 다시 채운다**

## 7단계: PR 갱신 및 자동 검사 확인

```bash
git add claude.md .claude/skills .claude/settings.json .claude/hooks app/blog/posts evidence/week-01__weekly-pr.md
git commit -m "1주차: claude.md, create-post 스킬, Pre/PostToolUse Hook 추가"
git push
```

- 자동 검사(`npm test`)와 AI 리뷰를 확인한다
- self-check 실패 시 `gh run view <run-id> --job <job-id> --log`로 원인을 직접 본다. "채점 Workflow/challenge.json 변경 금지" 에러면 `git merge origin/main` 후 재push
- PR의 "Files changed"에서 새로 추가한 파일(SKILL.md, settings.json, hooks 스크립트 등)이 전부 diff에 잡히는지 확인
- 보완 요청이 오면 같은 PR에 수정 커밋을 push하고, 없으면 evidence의 "## 리뷰 반영"에 "지적 없음"을 기록

---

## 근거형 질문 3개 — 답변 방향 메모

1. **claude.md에 필요한 정보**: package.json scripts 실제값 + README/challenge.json에서 확인한 금지 경로 목록 → 작성한 claude.md 본문 자체가 근거
2. **PreToolUse/PostToolUse 구분 기준**: 공식 문서(`https://code.claude.com/docs/en/hooks#exit-code-output`)의 실행 시점 설명 + 3·4단계에서 설치한 두 Hook 스크립트 경로 + 각각 발동한 로그
3. **Hook이 못 막은 실수**: 실제로 겪은 우회/누락 사례가 있으면 그 로그와 재현 절차. 없으면 "아직 확인 못한 한계"로 선택 근거 섹션과 연결해 정직하게 명시 (예: 이 Hook은 `Edit`/`Write` 도구만 매칭하므로 Bash로 파일을 직접 수정하면 걸리지 않는다는 한계)

---

## 증거 체크리스트 — 무엇을, 왜, 어떻게 남기는가

missions/README.md의 "필수 제출 증거" 5개 항목을 실제로 무엇을 캡처해야 채워지는지 풀어놓은 목록. self-check는 evidence 파일의 존재·글자수·5개 섹션 제목 유무만 기계적으로 검사하지만, **사람 리뷰(AI 리뷰)는 "통과만 확인했는지"를 본다** — 그래서 아래 항목은 결과 문장뿐 아니라 그 결과가 나온 실행 로그·캡처까지 요구한다.

### 1. claude.md 5항목

- **남길 것**: 완성된 claude.md 파일 자체 (커밋에 포함되면 자동으로 증거가 된다)
- **추가로 남길 것**: 1단계 마지막에 실행한 `npm run typecheck`, `npm test`의 **실제 출력 전문**. "통과했다"는 문장만 쓰지 말고 터미널 출력을 그대로 evidence에 복붙한다
- **왜**: claude.md에 적은 명령이 "AI가 만들어준 문장을 그대로 둔 것"이 아니라 직접 실행해서 확인한 것임을 증명하기 위함 (missions 통과 기준 1번)

### 2. 스킬과 그 산출물

- **남길 것**: `.claude/skills/create-post/SKILL.md` 파일 자체
- **추가로 남길 것**: 스킬을 실제로 실행해서 만든 새 글 파일(`app/blog/posts/<slug>.md`)이 커밋에 함께 있어야 한다 — 스킬 정의만 있고 산출물이 없으면 "만들어진 것을 확인 안 했다"는 지적을 받는다 (missions 통과 기준 2번)
- **추가로 남길 것**: `npm run dev` 실행 후 `/blog` 목록에 새 글이 올바른 순서로 보이는 화면 스크린샷, 또는 브라우저 대신 `curl localhost:3000/blog`로 받은 HTML에 글 제목이 포함된 걸 확인한 로그

### 3. PreToolUse 차단 로그 — 가장 중요, 빠뜨리기 쉬움

- **남길 것**: 금지 경로(예: `challenge.json`)를 고쳐달라고 **일부러 요청**했을 때, Hook이 exit code 2로 막는 순간의 **원문 그대로의 stderr 메시지**와 Claude Code 화면에 뜨는 거부 표시
- **어떻게**: 터미널 스크린샷 또는 텍스트 로그 복사. "Hook이 정상 작동했다"는 서술만 쓰면 근거가 아니라 주장이 된다
- **왜**: missions 통과 기준에 "통과만 확인한 것이 아님을 알 수 있다"고 명시되어 있다 — 정상 동작(차단 안 함)만 보여주면 이 기준을 못 채운다. 반드시 실패(차단됨) 케이스를 보여줘야 한다

### 4. PostToolUse 자동 실행 로그 — 정상/실패 두 케이스 다 필요

- **남길 것 (정상)**: `.ts`/`.tsx` 파일을 정상적으로 수정했을 때, Hook이 자동으로 `npm run typecheck`를 실행해 통과하는 로그
- **남길 것 (실패)**: 일부러 타입 에러가 나는 수정을 했을 때, Hook이 에러를 잡아내는 로그. 그 에러가 Claude에게 다시 전달되어 스스로 고치는 흐름까지 있으면 더 강한 증거가 된다
- **왜**: "무엇을 자동화했는지 설명"만으로는 부족하고, "실제로 실행된 로그"가 있어야 한다 (missions 통과 기준 4번)

### 5. 근거형 질문 1~3 답변

- **형식**: 질문마다 "최초 판단 → 연결한 코드/로그 → 검증 후 답변" 3단 구조 (공통 제출 계약 4번)
- **연결할 것**: 최초 판단은 prep-questions.md에 이미 적어둔 답, 연결할 코드/로그는 위 1~4번에서 남긴 실제 파일 경로와 로그, 검증 후 답변은 그 로그를 보고 최종적으로 확정한 결론

### 마지막: 리뷰 반영 — 제출은 끝이 아니라 시작

- PR을 올린 뒤 자동 리뷰나 self-check 결과가 달리면, **그 즉시** evidence의 "## 리뷰 반영"에 다시 들어가 기록한다
- 지적이 없으면 "지적 없음", 있으면 무엇을 어떻게 고쳤는지와 수정 커밋을 적는다
- 이 항목을 빈칸으로 두는 게 다른 dingco 챌린지들(backend-resume, jpa)에서 가장 많이 반복된 지적이었다 — 1주차부터 놓치지 않는다
