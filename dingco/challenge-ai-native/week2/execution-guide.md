# 2주차 실행 가이드

prep-questions는 이번 라운드는 건너뛰고 실행부터 진행한다. 대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18, 브랜치는 이미 만들어진 `submit/week-02__weekly-pr`를 이어서 쓴다 (main 대비 diff는 `evidence/week-02__weekly-pr.md` 하나뿐인 상태, main의 채점 워크플로우 변경은 base 브랜치 기준으로 자동 적용되므로 별도 반영 불필요 — 확인 완료).

미션 원문: missions/README.md 53~85행 (Week 2. 이슈에서 검증된 PR까지)

## 이번 주 선택 기능: 글 상세 페이지 이전/다음 글 네비게이션

`app/blog/[slug]/page.tsx`에 이전/다음 글로 이동하는 링크를 추가한다. `app/blog/posts.ts`의 `getPosts()`가 이미 `publishedAt` 기준 내림차순 정렬 배열을 반환하므로(24행), 현재 글의 배열 인덱스를 찾아 앞뒤 항목을 이전/다음으로 쓰면 된다. 아래 가이드의 `<기능>`, `<이슈번호>-<슬러그>` 자리는 이 기능 기준으로 다음처럼 채운다:

- 슬러그: `post-navigation`
- feature 브랜치명 예시: `feature/12-post-navigation` (12는 예시, 실제 이슈 번호로 교체)
- 테스트 파일명: `tests/post-navigation.spec.ts`
- 인수 조건 후보:
  - [ ] 글 상세 페이지에 "다음 글"/"이전 글" 링크가 보인다 (있을 때만)
  - [ ] 다음 글 링크 클릭 시 해당 글 상세로 이동한다
  - [ ] 목록의 마지막 글에서는 "다음 글" 링크가 없고, 첫 글에서는 "이전 글" 링크가 없다

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

## 1단계: 이슈 만들기

기능은 위에서 정한 "글 상세 페이지 이전/다음 글 네비게이션"으로 진행한다.

```bash
gh issue create --repo GaeChwiPpo/challenge-ai-native-2026-08-wlghsp-r18 \
  --title "글 상세 페이지에 이전/다음 글 네비게이션 추가" \
  --body "$(cat <<'EOF'
## 문제
글 상세 페이지(app/blog/[slug]/page.tsx)에서 다른 글로 이동하려면 다시 /blog 목록으로 나가야 한다. 인접한 글로 바로 이동할 수 있는 링크가 없다.

## 인수 조건
- [ ] 글 상세 페이지에 "다음 글"/"이전 글" 링크가 보인다 (있을 때만)
- [ ] 다음 글 링크 클릭 시 해당 글 상세로 이동한다
- [ ] 목록의 마지막 글에서는 "다음 글" 링크가 없고, 첫 글에서는 "이전 글" 링크가 없다
EOF
)"
```

출력에서 이슈 번호(`#N`)를 받는다. 이 번호가 브랜치명(`feature/<번호>-post-navigation`)·PR 본문에 그대로 쓰인다.

## 2단계: feature 브랜치 만들고 구현

```bash
git checkout -b feature/<이슈번호>-post-navigation
```

구현 지점은 두 파일이다.

### `app/blog/posts.ts`

파일 맨 아래(`parsePost` 함수 뒤)에 헬퍼를 추가한다:

```ts
export function getAdjacentPosts(slug: string): { previous: Post | null; next: Post | null } {
  const posts = getPosts()
  const index = posts.findIndex((post) => post.slug === slug)
  if (index === -1) return { previous: null, next: null }

  return {
    previous: index + 1 < posts.length ? posts[index + 1] : null,
    next: index > 0 ? posts[index - 1] : null,
  }
}
```

**방향 정의 근거**: `getPosts()`는 `publishedAt` **내림차순**(최신순) 정렬이다(24행 `left.publishedAt < right.publishedAt ? 1 : -1`). 배열 인덱스 0이 가장 최신 글이므로:
- 배열 인덱스가 더 큰 쪽(`index + 1`) = 더 오래된 글 = "이전 글"
- 배열 인덱스가 더 작은 쪽(`index - 1`) = 더 최신 글 = "다음 글"

이 방향은 임의로 바꿔도 되지만, 한번 정하면 코드·테스트·UI 문구에서 일관되게 써야 한다.

### `app/blog/[slug]/page.tsx`

import 줄에 `Link`와 `getAdjacentPosts`를 추가한다:

```ts
import Link from 'next/link'
import { getPost, getPosts, getAdjacentPosts } from '../posts'
```

`BlogPost` 함수 안, `post` 조회 직후에 추가:

```ts
const { previous, next } = getAdjacentPosts(params.slug)
```

`</article>` 다음에 형제 요소로(또는 전체를 `<>...</>` 프래그먼트로 감싸고) 네비게이션 블록을 추가한다:

```tsx
<nav className="mt-8 flex justify-between text-sm">
  {previous ? (
    <Link href={`/blog/${previous.slug}`}>← 이전 글: {previous.title}</Link>
  ) : (
    <span />
  )}
  {next ? (
    <Link href={`/blog/${next.slug}`}>다음 글: {next.title} →</Link>
  ) : (
    <span />
  )}
</nav>
```

### 구현 후 검증

```bash
npm run typecheck
npm run dev
```

브라우저에서 `/blog/hello-ai-native`, `/blog/context-is-everything` 등 각 글을 열어 이전/다음 링크가 올바른 글을 가리키는지, 첫 글/마지막 글에서 해당 없는 쪽 링크가 안 보이는지 확인한다.

에이전트로 기능을 구현한다. 구현하면서 아래를 메모해둔다 (3단계 docs/agent-workflow.md에 그대로 씀):
- 이번 주 쓴 도구: 에이전트 이름/버전 + 외부 도구(GitHub CLI `gh`, MCP 등)
- 읽기 전용으로 충분했던 지점 (예: `posts.ts`/`page.tsx` 읽기, 이슈 조회)
- 쓰기 권한이 필요했던 지점 (예: `posts.ts`/`page.tsx` Edit, `gh issue create`)
- 사람 승인이 필요했던 지점 (예: 이슈 생성 확정, merge 실행, PR push)

## 3단계: Playwright 시나리오 — 구현 전 실패(RED) 먼저 남긴다

TDD의 RED→GREEN 절차다. 기능을 구현하기 전에 먼저 테스트를 작성하고 실행해서 "기능이 없으면 확실히 실패한다(RED)"를 증거로 남긴 뒤, 구현하고 나서 같은 테스트가 "통과한다(GREEN)"를 남긴다. GREEN만 있으면 이 테스트가 실제로 기능을 검사하는지, 아니면 뭘 해도 통과하는 빈 테스트인지 구분이 안 되므로 RED가 반드시 먼저 있어야 한다 (missions/README.md 63행, 83행).

기존 글 3개의 `publishedAt`을 확인하면 (`app/blog/posts/*.md` frontmatter), `getPosts()`의 내림차순 정렬 결과는 다음과 같다:

1. `ai-native-challenge-intro` (2026-09-10, 가장 최신)
2. `hello-ai-native` (2026-09-02)
3. `context-is-everything` (2026-09-01, 가장 오래됨)

즉 `hello-ai-native` 글 기준으로 "다음 글"은 `ai-native-challenge-intro`, "이전 글"은 `context-is-everything`이다. 첫 글(`ai-native-challenge-intro`)에는 "다음 글" 링크가 없어야 하고, 마지막 글(`context-is-everything`)에는 "이전 글" 링크가 없어야 한다.

`tests/post-navigation.spec.ts` 작성:

```ts
import { expect, test } from '@playwright/test'

test('가운데 글에서 이전/다음 글 링크가 모두 보이고 클릭하면 이동한다', async ({ page }) => {
  await page.goto('/blog/hello-ai-native')

  const nextLink = page.getByRole('link', { name: /다음 글.*AI 네이티브 챌린지 소개/ })
  await expect(nextLink).toBeVisible()
  await nextLink.click()
  await expect(page).toHaveURL(/\/blog\/ai-native-challenge-intro$/)

  await page.goto('/blog/hello-ai-native')
  const prevLink = page.getByRole('link', { name: /이전 글.*컨텍스트가 전부다/ })
  await expect(prevLink).toBeVisible()
  await prevLink.click()
  await expect(page).toHaveURL(/\/blog\/context-is-everything$/)
})

test('가장 최신 글에는 다음 글 링크가 없다', async ({ page }) => {
  await page.goto('/blog/ai-native-challenge-intro')
  await expect(page.getByRole('link', { name: /다음 글/ })).toHaveCount(0)
})

test('가장 오래된 글에는 이전 글 링크가 없다', async ({ page }) => {
  await page.goto('/blog/context-is-everything')
  await expect(page.getByRole('link', { name: /이전 글/ })).toHaveCount(0)
})
```

이 파일을 **2단계 구현(posts.ts, page.tsx 수정) 전에** 먼저 저장하고 실행한다:

```bash
npx playwright install chromium   # 최초 1회
npm run e2e -- tests/post-navigation.spec.ts
```

아직 이전/다음 글 링크 자체가 없으므로 `getByRole('link', { name: /다음 글.../ })`를 못 찾아 타임아웃 실패가 난다. **이 실패 출력 전문(터미널 로그 전체, 어떤 selector를 못 찾았는지 나오는 부분 포함)을 그대로 저장해둔다** — evidence에 복붙할 RED 증거다.

## 4단계: 기능 구현 후 통과(GREEN) 확인

2단계에서 설명한 `posts.ts`의 `getAdjacentPosts` 헬퍼와 `page.tsx`의 네비게이션 링크를 구현한 뒤, 같은 테스트를 다시 돌린다:

```bash
npm run e2e -- tests/post-navigation.spec.ts
npm test
```

이번엔 3개 테스트가 모두 통과해야 한다. **통과 출력 전문**을 RED 로그와 나란히 저장한다 — 같은 테스트 파일, 같은 명령으로 실패→통과가 바뀐 것이 evidence에서 한눈에 비교돼야 한다.

## 5단계: feature 브랜치를 submit 브랜치로 병합

미션 규칙상 feature→main PR은 따로 열지 않는다. 채점 대상 PR은 `submit/week-02__weekly-pr`가 여는 PR 하나뿐이다.

```bash
git checkout submit/week-02__weekly-pr
git merge feature/<이슈번호>-post-navigation
```

머지 커밋이 생기면 그대로 두고(별도 squash 불필요), feature 브랜치는 지우지 않는다 — PR 본문에서 이슈·브랜치·PR 연결을 추적할 때 브랜치명이 남아 있어야 확인 가능하다.

## 6단계: docs/agent-workflow.md 작성

`docs/` 폴더가 없으면 새로 만든다.

```bash
mkdir -p docs
```

`docs/agent-workflow.md`에 아래 내용을 채운다. 이번 라운드 실제 흐름 기준(코드는 본인이 직접 타이핑, Claude Code는 이슈 생성 명령 작성·구현 방법 설명·가이드 문서화를 맡음)으로 이미 채워뒀으니 그대로 쓰거나 실제와 다른 부분만 고친다:

```markdown
# 에이전트 운영 기록

## Week 2 — 외부 도구 권한 경계

- 사용 에이전트: Claude Code (대화형, 코드 직접 수정 없이 설명·가이드 제공 역할로 한정)
- 사용 외부 도구: gh CLI (`gh issue create`로 이슈 #2 생성)
- 읽기 전용으로 충분했던 지점:
  - `app/blog/posts.ts`, `app/blog/[slug]/page.tsx` 기존 코드 구조 파악 (Claude가 Read로 확인 후 구현 방법만 설명)
  - `git log`, `git status`로 브랜치·커밋 상태 확인
  - `gh issue create` 실행 결과(이슈 번호) 확인
- 쓰기 권한이 필요했던 지점:
  - `gh issue create` — 실제 GitHub 이슈 생성 (Claude가 명령을 작성했고, 실행은 본인이 직접 함)
  - `app/blog/posts.ts`, `app/blog/[slug]/page.tsx`, `tests/post-navigation.spec.ts` 코드 작성 — 전부 본인이 직접 타이핑, Claude는 어디에 무엇을 쓸지 설명만 함
  - `git commit`, `git merge`, `git push` — 전부 본인이 직접 실행
- 사람 승인이 필요했던 지점:
  - 이슈 제목/본문 확정 (Claude가 초안 작성 → 본인이 검토 후 실행)
  - feature → submit 브랜치 merge 시점과 커밋 메시지 (본인이 직접 결정)
  - PR push 여부
```

Week 1에서 이미 docs/agent-workflow.md를 만들었다면 새 섹션(`## Week 2 — ...`)으로 이어서 추가한다. 이번 저장소는 Week 1에서 claude.md만 다뤘으므로 이번이 첫 작성이다.

## 7단계: evidence/week-02__weekly-pr.md 채우기

현재 draft(evidence/week-02__weekly-pr.md)를 아래처럼 채운다. RED/GREEN 로그와 PR 링크는 3·4·8단계를 실제로 실행한 뒤 그 출력을 그대로 붙여넣는다 — 나머지는 지금까지 진행 상황 기준으로 미리 채운 초안이다.

```markdown
# 선택한 에이전트로 이슈에서 검증된 PR까지 한 바퀴 돌리기

PR: <8단계에서 push 후 생성된 PR 링크>

## 변경

- 이슈 #2 "글 상세 페이지에 이전/다음 글 네비게이션 추가" 생성, `feature/2-post-navigation` 브랜치에서 구현
- `app/blog/posts.ts`: `getAdjacentPosts(slug)` 헬퍼 추가. `getPosts()`의 `publishedAt` 내림차순 정렬 배열에서 현재 글의 인덱스를 찾아 이전/다음 글을 반환
- `app/blog/[slug]/page.tsx`: 글 상세 하단에 이전/다음 글 링크를 조건부 렌더링 (없는 쪽은 표시 안 함)
- `tests/post-navigation.spec.ts`: 가운데 글에서 이전/다음 링크 클릭 시 이동, 최신 글에서 다음 링크 없음, 가장 오래된 글에서 이전 링크 없음 — 3개 시나리오
- `feature/2-post-navigation`을 `submit/week-02__weekly-pr`로 merge

## 검증

- RED (구현 전, `npm run e2e -- tests/post-navigation.spec.ts`):
  ```
  <여기에 3단계 실패 출력 전문 붙여넣기>
  ```
- GREEN (구현 후, 같은 명령):
  ```
  <여기에 4단계 통과 출력 전문 붙여넣기>
  ```
- `npm test` 결과:
  ```
  <여기에 전체 검사(tsc --noEmit && next build) 출력 붙여넣기>
  ```

## 선택 근거

- 이전/다음 글 방향은 "다음 글 = 더 최신 글"로 정의함. 블로그에서 다음 글을 누르면 최신 글로 이동하는 게 자연스럽다고 판단
- `getPosts()`를 매번 다시 호출해 정렬하는 방식을 택함(캐싱 안 함) — 글 개수가 적어 성능 이슈가 없고, 기존 `getPost(slug)`도 같은 패턴(27~29행)이라 구조를 통일함
- 버린 대안: frontmatter에 `prev`/`next` slug를 수동으로 적는 방식 — 글이 추가될 때마다 앞뒤 글을 직접 갱신해야 해서 `posts.ts` 자동 정렬 방식보다 유지보수 부담이 큼
- 확인하지 못한 한계: 글이 1개뿐일 때(이전/다음 모두 없음) 동작은 코드상 가능하나 실제 데이터로 검증 안 함

## 근거형 질문

**1. 사용한 외부 도구에서 읽기 전용으로 충분한 지점과 쓰기 권한이 필요한 지점을 어떻게 나눴나요?**

이슈 생성(`gh issue create`)처럼 실제 GitHub 상태를 바꾸는 동작만 쓰기로 분류하고, 코드 읽기·git 상태 확인은 전부 읽기 전용으로 뒀다. 코드 작성 자체는 에이전트에게 맡기지 않고 직접 타이핑해서, Edit 권한을 에이전트에 준 적이 없다 — docs/agent-workflow.md의 "쓰기 권한이 필요했던 지점" 목록이 그 경계선이다.

**2. 이슈·브랜치·PR 인수 조건을 제공한 뒤 에이전트의 결과물이 어떻게 달라졌나요?**

<이슈 #2의 인수 조건 3개(링크 표시/클릭 이동/양 끝 처리)를 미리 정해두고 구현했더니 무엇이 달라졌는지 — 예: 처음부터 "없을 때 렌더링 안 함" 케이스를 빠뜨리지 않고 짤 수 있었는지 등을 실제 경험 기준으로 채운다>

**3. Playwright 시나리오가 이 기능의 무엇을 보장하고, 무엇은 여전히 보장하지 못하나요?**

보장하는 것: 가운데 글에서 양쪽 링크가 다 보이고 클릭하면 정확한 slug로 이동하는 것, 양 끝 글에서 해당 없는 링크가 안 보이는 것. 보장하지 못하는 것: 글이 1개만 있을 때 동작(테스트 데이터에 없음), frontmatter `publishedAt` 형식이 깨졌을 때의 동작.

## 리뷰 반영

자동 리뷰 수신 전
```

## 8단계: PR 갱신 및 자동 검사 확인

```bash
git add docs/agent-workflow.md app/blog/posts.ts "app/blog/[slug]/page.tsx" tests/post-navigation.spec.ts evidence/week-02__weekly-pr.md
git commit -m "2주차: 글 상세 이전/다음 네비게이션과 이슈-브랜치-PR 흐름, Playwright RED/GREEN 증거 추가"
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
