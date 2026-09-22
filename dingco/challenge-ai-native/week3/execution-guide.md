# 3주차 실행 가이드

prep-questions는 이번 라운드도 건너뛰고 실행부터 진행한다. 3주차는 개념 확인보다 "실제로 리뷰를 돌리고, 실패를 재현하고, 신호를 관찰한 로그"가 핵심이라 사전 질문보다 실행이 먼저다.

대상 저장소는 challenge-ai-native-2026-08-wlghsp-r18, 브랜치는 이미 만들어진 `submit/week-03__weekly-pr`를 이어서 쓴다 (working tree clean 확인 완료, evidence/week-03__weekly-pr.md draft도 이미 존재).

미션 원문: missions/README.md 87~121행 (Week 3. AI 리뷰와 실패 복구)

## 이번 주 미션 구조

네 가지를 각각 증거로 남겨야 한다:

1. AI 리뷰로 지적 1개(또는 검증된 정상 판단 1개) 찾기
2. 의도적 실패 1개 재현 → 원인 → 수정 → 통과
3. 운영 신호 1개 정의 (정상/경고 기준을 숫자 또는 조건으로)
4. docs/agent-workflow.md에 이번 주 에이전트·사람 경계 기록

이 네 가지는 순서대로 할 필요는 없지만, 2번(실패 재현)이 1번(AI 리뷰)에서 나온 지적을 재료로 쓰면 두 항목이 자연스럽게 연결되어 evidence 작성이 쉬워진다. 아래 가이드는 그 순서로 진행한다.

## 0단계: 시작 전 확인

```bash
cd /Users/jihochoi/Documents/study/dingco/challenge-ai-native-2026-08-wlghsp-r18
git branch --show-current   # submit/week-03__weekly-pr 이어야 함
git status                  # clean 이어야 함
npm ci
npm test                    # tsc --noEmit && next build, 시작 전 기준선 확보
npm run e2e                 # 기존 tests/blog-navigation.spec.ts, tests/post-navigation.spec.ts 모두 통과 확인
```

## 1단계: AI 리뷰로 지적 찾기

week1~2에서 만든 코드(`app/blog/posts.ts`, `app/blog/[slug]/page.tsx`) 전체를 에이전트에게 리뷰시킨다. 예시 프롬프트:

> app/blog/posts.ts와 app/blog/[slug]/page.tsx를 리뷰해줘. 버그, 엣지 케이스 누락, 잘못된 가정이 있는지 찾아줘.

리뷰에서 나올 가능성이 높은 후보들 (코드를 미리 읽고 확인한 것 — 실제로 무엇이 나오는지는 리뷰를 실행해서 확정한다):

- `parsePost`의 정규식이 frontmatter를 못 찾으면(`match`가 null) `meta`가 빈 객체가 되어 `publishedAt`이 빈 문자열(`''`)로 들어간다. 이 글은 정렬 시 문자열 비교(`'' < '2026-09-01'`)에서 가장 오래된 글보다도 뒤로 밀리는데, 이게 의도한 동작인지 방어가 없다.
- `getPosts()`가 매 호출마다 파일시스템을 다시 읽고 정렬한다. `getPost(slug)`, `getAdjacentPosts(slug)` 모두 내부적으로 `getPosts()`를 다시 호출하므로 `BlogPost` 컴포넌트 한 번 렌더링에 파일시스템 읽기가 최소 2회(9~10행) 발생한다. 글 개수가 적어 지금은 문제없지만, 캐싱 없음이 의도적 선택인지 리뷰에서 지적될 수 있다.
- `getAdjacentPosts`가 `index === -1`(존재하지 않는 slug)일 때 `{previous: null, next: null}`을 반환하는데, 호출부(`page.tsx`)에서는 이미 `getPost`가 `undefined`면 `notFound()`로 빠지므로 이 분기는 실제로 도달하지 않는 죽은 방어 코드일 수 있다.

리뷰 결과를 받으면:
- 채택한 지적 1개는 2단계(의도적 실패 재현)의 재료로 쓴다.
- 채택하지 않은 지적이 있으면 왜 보류했는지 근거를 남긴다 (예: "글 개수가 3개뿐이라 캐싱 부재는 지금 단계에서 실제 문제가 아니다").

리뷰 응답 전문을 저장해둔다 — evidence의 "AI 리뷰 결과" 항목에 그대로 인용한다.

## 2단계: 의도적 실패 재현 (RED) → 원인 파악 → 수정 (GREEN)

이 단계는 이미 실제로 겪은 사례로 대체한다 — 코드에 버그를 일부러 심을 필요 없이, 0단계 검증 중 `npm run e2e`가 3개 테스트에서 실제로 실패했다. 실패가 "환경 문제"였다는 점이 오히려 이번 미션 취지(실패 재현·원인·복구)에 더 잘 맞는다.

### 실패 재현 (RED, 실제로 발생한 로그)

```bash
npm run e2e
```

```
Running 5 tests using 4 workers

  ✓  1 [chromium] › post-navigation.spec.ts:18:5 › 가장 최신 글에는 다음 글 링크가 없다 (3.1s)
  ✓  5 [chromium] › post-navigation.spec.ts:23:5 › 가장 오래된 글에는 이전 글 링크가 없다 (1.1s)
  ✘  3 [chromium] › post-navigation.spec.ts:3:5 › 가운데 글에서 이전/다음 글 링크가 모두 보이고 클릭하면 이동한다 (7.8s)
  ✘  2 [chromium] › blog-navigation.spec.ts:5:5 › 홈에서 글 목록으로 이동한다 (8.1s)
  ✘  4 [chromium] › blog-navigation.spec.ts:14:5 › 글 상세로 들어가면 본문이 보인다 (30.0s)

  1) blog-navigation.spec.ts:5:5 › 홈에서 글 목록으로 이동한다
    Error: Timed out 5000ms waiting for expect(locator).toBeVisible()
    Locator: getByRole('heading', { name: 'AI Native 실습 블로그' })
    Expected: visible
    Received: <element(s) not found>

  2) blog-navigation.spec.ts:14:5 › 글 상세로 들어가면 본문이 보인다
    Test timeout of 30000ms exceeded.
    Error: locator.click: Test timeout of 30000ms exceeded.
    waiting for getByRole('link', { name: 'AI Native 챌린지를 시작합니다' })

  3) post-navigation.spec.ts:3:5 › 가운데 글에서 이전/다음 글 링크가 모두 보이고 클릭하면 이동한다
    Error: Timed out 5000ms waiting for expect(locator).toBeVisible()
    Locator: getByRole('link', { name: /다음 글.*AI 네이티브 챌린지 소개/ })
    Expected: visible
    Received: <element(s) not found>

  3 failed
  2 passed (31.2s)
```

세 실패 모두 "특정 텍스트(제목·링크)를 못 찾음" 패턴이었다. 반면 통과한 2개는 "링크가 없음"을 확인하는 테스트라 텍스트 매칭이 없었다 — 이 대비가 원인을 좁히는 첫 단서였다.

### 원인 조사 과정

1. 브라우저로 `http://localhost:3000`을 직접 열어보니 제목·링크가 정상적으로 보였다 → 코드 문제는 아니라고 판단, 타이밍 문제 가설
2. dev 서버를 띄운 채로 `npm run e2e`를 다시 돌려도 같은 실패 → 타이밍 문제 가설 기각
3. `playwright.config.ts`를 확인 — `webServer.command`가 `npm run build && npm run start`라 Playwright는 dev 서버가 아니라 프로덕션 빌드 서버로 테스트한다는 걸 확인. 다만 `reuseExistingServer: !process.env.CI`라 로컬에서는 3000번 포트에 이미 떠 있는 서버를 재사용하려 시도할 수 있다는 점에 주목
4. `lsof -i :3000` → 리스닝 프로세스가 안 잡혀 혼란(도커 컨테이너라 호스트에 안 보임)
5. `docker ps` → **`challenge-backend-resume-2026-08-wlghsp-r17-grafana-1`(다른 챌린지에서 띄운 Grafana 컨테이너)가 3000번 포트를 점유** 중임을 확인. `npm run build && npm run start`가 이미 점유된 포트라 새 서버를 못 띄우고, `reuseExistingServer` 설정 때문에 기존 3000번 응답(=Grafana)을 그대로 재사용해 테스트가 엉뚱한 화면을 상대로 실행되고 있었다

### 원인

Next.js 앱의 코드 문제가 아니라, **다른 프로젝트(backend-resume 챌린지)의 Docker 컨테이너가 같은 포트(3000)를 먼저 점유**해서 Playwright가 그 컨테이너(Grafana)를 대상으로 테스트를 실행한 것이었다. `reuseExistingServer: !process.env.CI` 설정은 로컬 반복 실행 속도를 위한 것이지만, 포트에 떠 있는 게 우리 앱이 맞는지는 검증하지 않는다는 게 이번에 드러난 맹점이다.

### 수정 (복구)

```bash
docker stop challenge-backend-resume-2026-08-wlghsp-r17-grafana-1
lsof -i :3000   # LISTEN 상태 프로세스 없음을 확인
```

### 통과 확인 (GREEN, 실제로 발생한 로그)

```bash
npm run e2e
```

```
Running 5 tests using 4 workers

  ✓  1 [chromium] › post-navigation.spec.ts:18:5 › 가장 최신 글에는 다음 글 링크가 없다 (576ms)
  ✓  2 [chromium] › blog-navigation.spec.ts:5:5 › 홈에서 글 목록으로 이동한다 (840ms)
  ✓  3 [chromium] › post-navigation.spec.ts:3:5 › 가운데 글에서 이전/다음 글 링크가 모두 보이고 클릭하면 이동한다 (739ms)
  ✓  4 [chromium] › blog-navigation.spec.ts:14:5 › 글 상세로 들어가면 본문이 보인다 (832ms)
  ✓  5 [chromium] › post-navigation.spec.ts:23:5 › 가장 오래된 글에는 이전 글 링크가 없다 (136ms)

  5 passed (9.4s)
```

같은 5개 테스트, 같은 명령으로 실패(3개)→통과(5개)가 뒤집힌 것이 RED/GREEN 쌍으로 남았다.

## 3단계: 운영 신호 정의

2단계 사례에서 자연스럽게 신호 하나가 나온다 — **테스트 실행 전 대상 포트 점유 상태**다:

- **신호**: `npm run e2e` 실행 직전 `lsof -i :3000` (또는 `docker ps`)로 확인한 3000번 포트 점유 프로세스
- **정상 기준**: 3000번 포트에 LISTEN 상태 프로세스가 없거나(Playwright가 새로 띄움), 있다면 그게 이 저장소의 Next.js 프로세스임이 확인됨
- **경고 기준**: 3000번 포트에 다른 프로젝트의 프로세스(이번처럼 Grafana 등)가 LISTEN 중 — 이 경우 `npm run e2e` 결과를 신뢰할 수 없으므로 테스트 실행 전에 반드시 해소해야 함

또는 `npm test`/`npm run e2e`의 CI 결과 자체를 신호로 삼아도 된다:

- **신호**: GitHub Actions의 `npm test`(selfCheck) 통과율
- **정상 기준**: 해당 커밋의 워크플로우 run이 성공(success)으로 종료
- **경고 기준**: 같은 브랜치에서 연속 2회 이상 실패, 또는 `next build` 단계에서 타입 에러 발생

둘 중 하나를 고르거나 다른 신호를 정해도 되는데, **정상/경고를 숫자나 명확한 조건으로** 적는 게 통과 기준(119행)이다. "잘 지켜보겠다" 같은 표현은 피한다.

## 4단계: docs/agent-workflow.md 갱신

week2까지 이미 섹션이 쌓여 있을 것이다. `## Week 3 — AI 리뷰와 실패 복구`로 이어서 추가한다:

```markdown
## Week 3 — AI 리뷰와 실패 복구

- 사용 에이전트: Claude Code (1단계 코드 리뷰 수행, 2단계 `npm run e2e` 실패 로그 해석·원인 후보 좁히기·다음 확인 명령 제시, 3단계 신호 후보 제시)
- AI 리뷰가 맡은 일: app/blog/posts.ts, app/blog/[slug]/page.tsx 리뷰 → <채택한 지적> 발견
- 실패 복구에서 맡은 일: 실패 로그를 보고 "타이밍 문제 → dev 서버 상태 → playwright.config.ts 설정 → lsof/docker ps" 순으로 원인 후보를 좁혀가는 진단 절차를 단계별로 제시. 각 단계의 실제 실행과 결과 확인은 본인이 직접 함
- 사람이 확인한 경계:
  - 리뷰 지적 중 실제로 재현 가능한지 테스트로 검증 (리뷰 결과를 그대로 신뢰하지 않음)
  - `docker ps`로 실제 원인(Grafana 컨테이너)을 최종 확인하고 컨테이너 정지 여부 결정 — 다른 챌린지에서 쓰는 컨테이너라 끄기 전 직접 판단
  - 운영 신호의 정상/경고 기준값 결정
- 읽기 전용으로 충분했던 지점: 코드 리뷰, 로그 해석, `lsof`/`docker ps` 같은 진단 명령 실행
- 쓰기 권한이 필요했던 지점: `docker stop`으로 컨테이너 정지 — 실제 실행은 본인이 직접 함
```

실제 진행하면서 무엇을 에이전트에게 맡기고 무엇을 직접 결정했는지에 맞춰 구체적으로 고친다.

## 5단계: evidence/week-03__weekly-pr.md 채우기

이미 draft가 있다 (evidence/week-03__weekly-pr.md). 섹션별로 채운다:

- **변경**: 1, 3, 4단계에서 만든/고친 파일 목록과 각각 무엇을 왜 했는지. 2단계는 코드 변경이 아니라 환경 문제(포트 충돌) 재현·복구였다는 점을 명시한다
- **검증**: 2단계 RED/GREEN 로그(포트 충돌 전후 `npm run e2e` 결과), `npm test` 결과, AI 리뷰 응답 전문
- **선택 근거**: 포트 충돌을 회귀하지 않게 하려면 무엇을 더 할 수 있는지(예: `webServer` 설정에 포트 점유 사전 확인 스크립트 추가), 버린 대안, 확인 못한 한계 (예: 이번엔 Grafana였지만 다른 프로세스가 3000번을 잡는 경우는 아직 겪어보지 못함)
- **근거형 질문**: missions/README.md 111~113행 3문항 — AI 리뷰가 잡아준 지적 중 사람이 놓쳤을 만한 것/채택-보류 이유, 이번 실패(포트 충돌)를 테스트·리뷰·운영 신호 중 무엇이 가장 먼저 발견했어야 하는지, 운영 신호와 경보 기준
- **리뷰 반영**: 자동 리뷰 수신 전이면 "지적 없음" 대신 비워두고 8단계 이후 채운다

## 6단계: 커밋 및 push

```bash
git add docs/agent-workflow.md evidence/week-03__weekly-pr.md
# 1단계 AI 리뷰를 반영해 posts.ts 등을 실제로 고쳤다면 그 파일도 포함
git commit -m "3주차: AI 리뷰 지적 반영, 포트 충돌 실패 재현·복구, 운영 신호 정의"
git push
```

## 7단계: PR 자동 검사·AI 리뷰 확인

- 자동 검사(`npm test`)와 AI 리뷰 확인
- self-check 실패 시 `gh run view <run-id> --job <job-id> --log`로 원인 직접 확인
- 보완 요청이 오면 같은 PR에 수정 커밋 push, 없으면 evidence의 "## 리뷰 반영"에 "지적 없음" 기록

---

## 증거 체크리스트 — 무엇을, 왜 남기는가

missions/README.md 필수 제출 증거(101~107행) 기준.

### 1. AI 리뷰 결과와 채택·보류 이유

- **남길 것**: 리뷰 응답 전문 + 채택한 지적 1개(또는 검증된 정상 판단 1개)와 그 근거
- **왜**: 통과 기준(117행) — "리뷰 결과를 그대로 따르지 않고 코드·테스트 근거로 채택 또는 보류"가 확인돼야 한다. 리뷰를 그냥 받아쓰기만 하면 통과하지 못한다.

### 2. 의도적 실패 재현·원인·수정 후 통과

- **남길 것**: RED 로그(3개 실패), 원인 조사 과정과 결론(Grafana 컨테이너의 포트 점유), GREEN 로그(5개 통과) — 같은 명령(`npm run e2e`)의 전/후 결과이어야 한다
- **왜**: 통과 기준(118행) — "같은 조건으로 남아 있어 복구가 확인된다". 코드 버그가 아니라 환경 문제였다는 점도 정직하게 적는다 — 미션이 요구하는 건 "실패 재현·원인·복구"이지 "코드 버그"로 한정하지 않는다

### 3. 운영 신호와 정상·경고 기준

- **남길 것**: 신호 1개, 숫자 또는 명확한 조건으로 적은 정상/경고 기준
- **왜**: 통과 기준(119행) — 모호한 표현이 아니라 구체적 기준이어야 한다

### 4. 근거형 질문 1~3 답변

- **형식**: 최초 판단 → 연결한 코드/로그 → 검증 후 답변
- **연결할 것**: 1단계 리뷰 응답, 2단계 RED/GREEN 로그, 3단계 신호 정의를 그대로 인용

### 마지막: 리뷰 반영

- PR 제출은 끝이 아니라 시작. 리뷰/self-check 결과가 달리면 그 즉시 "## 리뷰 반영"을 다시 채운다
- 지적 없으면 "지적 없음", 있으면 무엇을 어떻게 고쳤는지와 수정 커밋을 적는다
