# 삭제된 파일을 git 히스토리에서 복원하기

## 한 줄 요약

`git rm`/`git commit`으로 지운 파일은 저장소에서 안 보일 뿐, 과거 커밋 안에는 그대로 남아있다. 필요하면 언제든 꺼내올 수 있다.

---

## 왜 걱정 없이 지워도 되는가

git은 파일을 "삭제"해도 실제로는 "이 커밋 시점부터는 이 파일이 없다"고 기록할 뿐, 그 이전 커밋들은 전혀 건드리지 않는다. 즉 삭제 커밋 자체가 "복원 지점"의 역할을 한다 — 그 커밋 **바로 이전** 상태를 보면 파일이 그대로 있다.

---

## 1. 언제 삭제됐는지 찾기

```bash
git log --oneline --diff-filter=D -- {파일 또는 폴더 경로}
```

`--diff-filter=D`는 "이 커밋에서 삭제(Delete)된 것만 보여줘"라는 뜻. 예:

```bash
git log --oneline --diff-filter=D -- docs/stitch_etude_auth_progress_ui/
# 20850cf Stitch UI 목업 자료 삭제 — 실제 프론트엔드에 반영 완료된 참고용 자료
```

이 커밋 해시(`20850cf`)가 "삭제한 순간"이다. 복원하려면 **이 커밋의 바로 이전**(`20850cf~1`, 물결표+1은 "한 커밋 전"이라는 뜻) 상태를 보면 된다.

---

## 2. 파일 하나만 복원 (가장 흔한 경우)

```bash
git checkout 20850cf~1 -- docs/stitch_etude_auth_progress_ui/login_etude/code.html
```

이 명령은 지정한 파일(또는 폴더 전체)만 그 시점 상태로 되돌려서, **지금 작업 중인 폴더에 다시 가져다 놓는다**. 나머지 파일들은 전혀 건드리지 않는다.

폴더째로 복원하고 싶으면:

```bash
git checkout 20850cf~1 -- docs/stitch_etude_auth_progress_ui/
```

복원된 파일은 일반적인 변경사항처럼 `git status`에 나타난다. 확인 후 `git add` + `git commit`으로 다시 커밋하면 된다.

---

## 3. 내용만 잠깐 들여다보고 싶을 때 (복원 없이)

파일을 다시 가져올 필요 없이 "그때 내용이 뭐였는지"만 보고 싶다면:

```bash
git show 20850cf~1:docs/stitch_etude_auth_progress_ui/login_etude/code.html
```

터미널에 그 시점의 파일 내용이 그대로 출력된다. 아무것도 복원되지 않으니 안전하게 확인용으로 써도 된다.

---

## 4. 폴더 전체를 별도 위치에서 통째로 살펴보고 싶을 때

지금 작업 폴더를 건드리지 않고, 그 시점의 프로젝트 전체를 별도 폴더에 펼쳐서 보고 싶을 때는 `git worktree`를 쓴다.

```bash
git worktree add /tmp/restore-test 20850cf~1
# /tmp/restore-test 에 그 시점 전체 프로젝트가 펼쳐진다

ls /tmp/restore-test/docs/stitch_etude_auth_progress_ui/
# 파일들 확인

# 다 보고 나면 정리
git worktree remove /tmp/restore-test --force
```

여러 파일을 한꺼번에 훑어보거나, 실제로 그 시점 코드를 실행까지 해봐야 할 때 유용하다.

---

## 정리 — 상황별로 뭘 쓸지

| 상황 | 명령 |
|---|---|
| 삭제된 파일/폴더를 다시 프로젝트에 가져오고 싶다 | `git checkout {삭제 커밋}~1 -- {경로}` |
| 내용만 잠깐 보고 싶다 (복원 안 함) | `git show {삭제 커밋}~1:{파일경로}` |
| 여러 파일을 한꺼번에 탐색/실행해보고 싶다 | `git worktree add {임시경로} {삭제 커밋}~1` |

---

## 참고 — 이 문서를 쓰게 된 배경

2026-07-03, `docs/stitch_etude_auth_progress_ui/`(Google Stitch로 만든 UI 디자인 목업, Phase 3/7 당시 참고용)를 실제 프론트엔드 구현이 끝난 뒤 정리했다. 삭제 당시 "git 히스토리에는 남아있으니 필요하면 복원 가능"이라고 했던 걸 실제로 검증해서 문서로 남김 — 삭제 커밋(`20850cf`) 이전(`20850cf~1`)에서 정상적으로 파일이 조회/복원됨을 확인했다.
