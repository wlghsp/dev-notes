# 터미널 단축키 (bash/zsh)

## 커서 이동

- `Ctrl + A` — 줄 맨 앞으로
- `Ctrl + E` — 줄 맨 끝으로
- `Option + →` (macOS) / `Alt + F` (Linux) — 한 단어 앞으로
- `Option + ←` (macOS) / `Alt + B` (Linux) — 한 단어 뒤로

## 삭제

- `Ctrl + U` — 커서 앞부분 전체 삭제
- `Ctrl + K` — 커서 뒷부분 전체 삭제
- `Ctrl + W` — 커서 앞 단어 삭제
- `Option + D` (macOS) / `Alt + D` (Linux) — 커서 뒤 단어 삭제

## 히스토리 / 검색

- `Ctrl + R` — 명령어 히스토리 역검색 (한 번 더 누르면 다음 매치)
- `!!` — 직전 명령어 재실행
- `!$` — 직전 명령어의 마지막 인자 재사용 (`mkdir foo` → `cd !$`)
- `Ctrl + P` / `Ctrl + N` — 이전/다음 히스토리 (↑/↓ 방향키와 동일)

## 작업 제어

- `Ctrl + C` — 현재 프로세스 강제 종료
- `Ctrl + Z` — 프로세스 일시정지 → `fg`로 복귀
- `Ctrl + D` — EOF / 쉘 종료

## 화면

- `Ctrl + L` — 화면 클리어 (스크롤 버퍼는 유지)
- `Ctrl + _` — 마지막 편집 취소 (undo)
- `Cmd + K` (macOS Terminal/iTerm2) — 화면 + 스크롤 버퍼 전체 클리어

## macOS 터미널 앱 설정 주의사항

`Option` 키를 메타 키로 사용하려면 터미널 앱에서 별도 설정이 필요하다.

**Terminal.app**: 환경설정 → 프로파일 → 키보드 → "Option을 Meta 키로 사용" 체크
**iTerm2**: Preferences → Profiles → Keys → Left/Right Option key를 `Esc+`로 변경

설정 전에는 `Option + →` 같은 단어 이동 단축키가 동작하지 않고, 특수문자(예: `∂`)가 입력된다.

**Ghostty**: 기본값으로 `Option`이 메타 키로 동작하므로 별도 설정 불필요.

## tmux 사용 시 주의

tmux prefix(`Ctrl + B`) 계열 단축키와 위 bash 단축키는 레이어가 다르므로 충돌하지 않음.
단, `Ctrl + B` 자체가 bash에서 "한 글자 뒤로"이므로, tmux 사용 중에는 해당 bash 단축키는 prefix로 소비됨.

## zsh 키 바인딩 모드

zsh 기본값은 **emacs 모드** — 위 단축키 전부 동작.
vi 모드(`bindkey -v`) 사용 시 Normal/Insert 모드 전환이 생기며 일부 단축키 동작이 달라짐.
