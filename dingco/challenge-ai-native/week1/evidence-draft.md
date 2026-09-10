# Week 1 증거 (초안)

challenge-ai-native 저장소의 evidence/week-01__weekly-pr.md에 옮겨 적을 초안. 이 초안을 보고 본문은 직접 다듬어서 채운다.

## 변경

- claude.md에 프로젝트 구조/실행 명령/코딩 규칙/금지 변경/완료 조건 채움
- create-post 스킬 추가 (블로그 글 생성 반복 작업)
- PreToolUse Hook: missions/, challenge.json, .github/workflows/ 수정 차단
- PostToolUse Hook: .ts/.tsx 수정 시 typecheck 자동 실행

## 검증

- `npm run typecheck`, `npm test` 통과 확인
- 스킬로 글 1개 생성 → `/blog` 목록에 정상 반영 (evidence/week-01/skill/)
- PreToolUse 차단: 처음엔 Hook이 안 먹혀서 원인 찾아보니 `*/challenge.json` 패턴이 상대경로(`challenge.json`)를 못 잡는 버그였음. 패턴 수정 후 재확인하니 정상 차단됨 (evidence/week-01/pretooluse/)
- PostToolUse: 정상 수정은 typecheck 통과, 일부러 타입 에러 낸 파일은 typecheck가 잡아냄 (evidence/week-01/posttooluse/)

## 선택 근거

- Hook은 별도 설치 없이 셸 스크립트로 구현 (Node는 이미 있어서)
- dispatcher 패턴(선택 확장)은 이번엔 스킵
- 한계: Edit/Write 말고 Bash로 직접 파일 고치면 이 Hook으로 못 막음. typecheck도 tsconfig 범위 밖 파일은 안 잡힘 (스크래치 경로에서 확인함)

## 근거형 질문

**1. claude.md에 반드시 있어야 하지만 README에는 없어도 되는 정보는?**

AI가 작업할 때마다 참조해야 하는 실행 가능한 규칙(정확한 명령어, 금지 경로, 완료 조건). README는 사람이 훑어보는 설명이고 claude.md는 AI가 지켜야 하는 제약이라고 봄. 실제로 claude.md에 적은 금지 경로 목록을 그대로 Hook 차단 패턴에 옮겨 썼다.

**2. PreToolUse로 막아야 하는 변경과 PostToolUse로 뒤처리하면 되는 변경을 어떻게 나눴나?**

되돌리기 힘든 건 사전 차단(PreToolUse), 고치면 되는 건 사후 감지(PostToolUse)로 나눴다. challenge.json은 PreToolUse가 애초에 Edit 자체를 막아서 파일이 바뀔 일이 없었고, 타입 에러는 일단 파일이 쓰인 다음 PostToolUse가 잡아냈다.

**3. Hook을 붙였는데도 막지 못한 실수는 무엇이고 다음에 어떻게 막을 계획인가?**

`*/challenge.json` 패턴이 슬래시 없는 상대경로를 못 잡아서 처음엔 Hook이 안 막았다. `echo '{"tool_input":{"file_path":"challenge.json"}}' | .claude/hooks/block-protected-paths.sh`로 재현해보니 exit 0(통과)이 나와서 스크립트 버그라는 걸 확인했다. 슬래시 없는 패턴도 추가해서 고쳤고 상대/절대경로 둘 다 재검증했다. 다음부턴 Hook 짤 때 파일 경로가 상대/절대 둘 다 들어올 수 있다고 가정하고 직접 테스트해봐야겠다.

## 리뷰 반영

- 자동 리뷰 수신 전
