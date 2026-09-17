# 2주차 실행 가이드

prep-questions는 이번 라운드는 건너뛰고 실행부터 진행한다. 대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18, 브랜치는 이미 만들어진 `submit/week-02__weekly-pr`를 이어서 쓴다 (main 대비 diff는 `evidence/week-02__weekly-pr.md` 하나뿐인 상태, main의 채점 워크플로우 변경은 base 브랜치 기준으로 자동 적용되므로 별도 반영 불필요 — 확인 완료).

미션 원문: missions/README.md 53~85행 (Week 2. 이슈에서 검증된 PR까지)

---

## 0단계: 시작 전 확인

```bash
cd /Users/jihochoi/Documents/study/dingco/challenge-ai-native-2026-08-wlghsp-r18
git branch --show-current   # submit/week-02__weekly-pr 이어야 함
git status                  # clean 이어야 함
npm ci
npm test                    # tsc --noEmit && next build, 시작 전 기준선 확보
find docs -type f 2>&1      # 아직 없으면 정상 (1주차엔 docs/ 없이 claude.md만 있었음)
```

## 1단계: 기능 정하고 이슈 만들기

기능은 `app/` 블로그 앱에 붙일 수 있고 Playwright 시나리오 1개로 고정 가능한 크기로 잡는다. 너무 크게 잡지 않는다 — 미션 통과 기준은 "작은 기능 + RED→GREEN 테스트"이지 규모가 아니다.

후보 예시 (택 1):
- 글 상세 페이지에 이전/다음 글 네비게이션 추가
- `/blog` 목록에 글 개수 표시
- 글 상세에 "목록으로" 돌아가기 링크 추가

```bash
gh issue create --repo GaeChwiPpo/challenge-ai-native-2026-08-wlghsp-r18 \
  --title "<기능 제목>" \
  --body "$(cat <<'EOF'
## 문제
<지금 뭐가 없거나 불편한지>

## 인수 조건
- [ ] <조건 1>
- [ ] <조건 2>
EOF
)"
```

출력에서 이슈 번호(`#N`)를 받는다. 이 번호가 브랜치명·PR 본문에 그대로 쓰인다.

## 2단계: feature 브랜치 만들고 구현

```bash
git checkout -b feature/<이슈번호>-<슬러그>
```

여기서 에이전트로 기능을 구현한다. 구현하면서 아래를 메모해둔다 (3단계 docs/agent-workflow.md에 그대로 씀):
- 이번 주 쓴 도구: 에이전트 이름/버전 + 외부 도구(GitHub CLI `gh`, MCP 등)
- 읽기 전용으로 충분했던 지점 (예: 코드 읽기, 이슈 조회)
- 쓰기 권한이 필요했던 지점 (예: 파일 Edit, `gh issue create`)
- 사람 승인이 필요했던 지점 (예: 이슈 생성 확정, merge 실행, PR push)

## 3단계: Playwright 시나리오 — 구현 전 실패(RED) 먼저 남긴다

`tests/<기능>.spec.ts` 파일을 먼저 작성한다. 이 시점엔 기능이 아직 없으므로 반드시 실패해야 한다.

```bash
npx playwright install chromium   # 최초 1회
npm run e2e -- tests/<기능>.spec.ts
```

**이 실패 출력 전문을 그대로 저장해둔다** (evidence에 복붙할 것). 통과 출력만 남기면 "테스트가 실제로 기능을 검사하는지 확인 안 됨"이라는 지적을 받는다 (Week 1의 PreToolUse 차단 로그와 같은 이유).

## 4단계: 기능 구현 후 통과(GREEN) 확인

```bash
npm run e2e -- tests/<기능>.spec.ts
npm test
```

두 출력 모두 전문을 저장한다.

## 5단계: feature 브랜치를 submit 브랜치로 병합

미션 규칙상 feature→main PR은 따로 열지 않는다. 채점 대상 PR은 `submit/week-02__weekly-pr`가 여는 PR 하나뿐이다.

```bash
git checkout submit/week-02__weekly-pr
git merge feature/<이슈번호>-<슬러그>
```

머지 커밋이 생기면 그대로 두고(별도 squash 불필요), feature 브랜치는 지우지 않는다 — PR 본문에서 이슈·브랜치·PR 연결을 추적할 때 브랜치명이 남아 있어야 확인 가능하다.

## 6단계: docs/agent-workflow.md 작성

`docs/` 폴더가 없으면 새로 만든다.

```bash
mkdir -p docs
```

`docs/agent-workflow.md`에 아래 항목을 채운다 (2단계에서 메모해둔 내용 사용):

```markdown
# 에이전트 운영 기록

## Week 2 — 외부 도구 권한 경계

- 사용 에이전트: <이름/버전>
- 사용 외부 도구: <gh CLI, MCP 등 — 실제 쓴 것만>
- 읽기 전용으로 충분했던 지점: <구체적으로>
- 쓰기 권한이 필요했던 지점: <구체적으로 — 어떤 명령/도구로 무엇을 썼는지>
- 사람 승인이 필요했던 지점: <구체적으로 — 어느 단계에서 사람이 직접 확인/실행했는지>
```

Week 1에서 이미 docs/agent-workflow.md를 만들었다면 새 섹션(`## Week 2 — ...`)으로 이어서 추가한다. 이번 저장소는 Week 1에서 claude.md만 다뤘으므로 이번이 첫 작성이다.

## 7단계: evidence/week-02__weekly-pr.md 채우기

현재 draft(evidence/week-02__weekly-pr.md)의 TODO 섹션을 아래 기준으로 채운다.

- `## 변경`: 이슈 번호, feature 브랜치명, 구현한 기능 요약
- `## 검증`: 3단계 RED 출력 전문 + 4단계 GREEN 출력 전문 + `npm test` 결과. "통과했다" 문장만 쓰지 말고 터미널 출력을 그대로 붙인다
- `## 선택 근거`: 이 기능/구현 방식을 고른 이유, 버린 대안, 아직 확인 못한 한계
- `## 근거형 질문`: 아래 3개에 답변 (형식은 최초 판단 → 연결한 코드/로그 → 검증 후 답변)
  1. 사용한 외부 도구에서 읽기 전용으로 충분한 지점과 쓰기 권한이 필요한 지점을 어떻게 나눴나요?
  2. 이슈·브랜치·PR 인수 조건을 제공한 뒤 에이전트의 결과물이 어떻게 달라졌나요?
  3. Playwright 시나리오가 이 기능의 무엇을 보장하고, 무엇은 여전히 보장하지 못하나요?
- `## 리뷰 반영`: 최초 제출은 "자동 리뷰 수신 전". PR에 리뷰/self-check 결과가 달리면 그 즉시 다시 채운다

## 8단계: PR 갱신 및 자동 검사 확인

```bash
git add docs/agent-workflow.md tests evidence/week-02__weekly-pr.md
git commit -m "2주차: 이슈-브랜치-PR 흐름과 Playwright RED/GREEN 증거 추가"
git push
```

- PR 본문에 이슈 번호, feature 브랜치명, 인수 조건을 명시적으로 연결해 남긴다 (통과 기준 2번: "이슈 번호와 브랜치명이 규칙대로 연결되고 PR 템플릿 항목이 빈칸 없이 채워져 있다")
- 자동 검사(`npm test`)와 AI 리뷰 확인
- self-check 실패 시 `gh run view <run-id> --job <job-id> --log`로 원인 직접 확인
- 보완 요청이 오면 같은 PR에 수정 커밋 push, 없으면 evidence의 "## 리뷰 반영"에 "지적 없음" 기록

---

## 증거 체크리스트 — 무엇을, 왜, 어떻게 남기는가

missions/README.md의 "필수 제출 증거" 4개 항목(68~71행) 기준.

### 1. 외부 도구·권한 범위

- **남길 것**: docs/agent-workflow.md 파일 자체
- **왜**: "GitHub 썼다" 수준이 아니라 구체적 도구·동작 단위로 적혀야 통과 기준(81행) 충족

### 2. 이슈·브랜치·PR 연결

- **남길 것**: 이슈 번호, `feature/<이슈번호>-<슬러그>` 브랜치명, PR 본문에 적은 연결 문장
- **왜**: 통과 기준(82행) — 브랜치명 규칙과 PR 템플릿 빈칸 여부를 본다

### 3. Playwright RED→GREEN — 가장 중요, 빠뜨리기 쉬움

- **남길 것**: 3단계의 실패 출력 전문 + 4단계의 통과 출력 전문, 둘 다
- **왜**: 통과 기준(83행)에 "구현 전 실패 출력과 구현 후 통과 출력이 모두 있어야" 한다고 명시. GREEN만 있으면 Week 1의 PreToolUse 케이스와 같은 이유로 "통과만 확인했다"는 지적을 받는다

### 4. 근거형 질문 1~3 답변

- **형식**: 최초 판단 → 연결한 코드/로그 → 검증 후 답변 (공통 제출 계약 4번)
- **연결할 것**: docs/agent-workflow.md의 권한 구분, 3·4단계 RED/GREEN 로그, PR 본문의 이슈 연결 내용을 그대로 인용

### 마지막: 리뷰 반영

- PR 제출은 끝이 아니라 시작. 리뷰/self-check 결과가 달리면 그 즉시 "## 리뷰 반영"을 다시 채운다
- 지적 없으면 "지적 없음", 있으면 무엇을 어떻게 고쳤는지와 수정 커밋을 적는다
- Week 1 evidence에서도 이 항목을 빈칸으로 뒀다가 지적받은 전례가 있다 — 처음부터 채워둔다
