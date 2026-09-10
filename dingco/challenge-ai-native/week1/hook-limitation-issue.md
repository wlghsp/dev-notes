# PreToolUse Hook이 처음엔 못 막았던 이유 — 패턴 매칭 버그

1주차 PreToolUse Hook(`block-protected-paths.sh`)을 만들고 실제로 위반 요청을 넣어 차단 로그를 남기려던 중, Hook이 아예 발동은 했는데 차단에는 실패하는 걸 직접 겪었다. 이 문서는 그 과정과 원인, 이게 미션이 의도한 함정인지, 어떻게 고쳤는지를 정리한다.

## 무슨 일이 있었는가

challenge-ai-native 세션에서 Claude에게 "challenge.json을 수정하는 Edit 도구를 실제로 호출해줘"라고 요청했다. 기대한 결과는 PreToolUse Hook이 exit code 2로 막는 것이었는데, 실제로는 `Update(challenge.json)`이 그대로 성공해서 파일이 수정됐다.

- 1차 시도: Claude가 스스로 CLAUDE.md 규칙을 읽고 "이건 하면 안 된다"며 도구 호출 자체를 안 함 — 이건 Claude의 판단으로 막힌 것이지 Hook이 작동한 증거가 아니었다
- 2차 시도: "도구 호출 자체는 시도해줘, Hook이 막을 거야"라고 요청 → 이번엔 Claude가 실제로 `Edit`을 호출했고, **Hook이 막지 못해 파일이 실제로 바뀌었다** (`"tampered": true` 추가됨, 이후 git checkout으로 원복)

## 원인

`.claude/hooks/block-protected-paths.sh`의 `case` 패턴이 이랬다.

```bash
case "$FILE_PATH" in
  */missions/*|*/challenge.json|*/.github/workflows/*)
    ...
    exit 2
    ;;
esac
```

`*/challenge.json` 패턴은 **경로에 슬래시(`/`)가 최소 하나 있어야** 매칭된다. 그런데 Claude Code가 `Edit` 도구에 넘기는 `file_path`가 항상 절대 경로인 게 아니었다 — 프로젝트 루트 파일은 **상대 경로**(`challenge.json`, 슬래시 없는 순수 파일명)로 넘어올 때가 있었다.

직접 재현:

```bash
echo '{"tool_input":{"file_path":"challenge.json"}}' | .claude/hooks/block-protected-paths.sh
echo "exit: $?"
# exit: 0   (통과 — 버그)

echo "{\"tool_input\":{\"file_path\":\"$(pwd)/challenge.json\"}}" | .claude/hooks/block-protected-paths.sh
echo "exit: $?"
# exit: 2   (정상 차단)
```

같은 파일인데 경로 형태(상대 vs 절대)에 따라 차단 여부가 갈렸다. 스크립트 로직 자체의 버그였지, Hook 메커니즘이나 `.claude/settings.json` 로드 문제가 아니었다 — 처음엔 "새로 만든 설정이라 세션에 아직 안 실려서 그런가" 하고 세션을 재시작해서 다시 시도했는데도 똑같이 뚫려서, 그제서야 스크립트 자체를 의심하고 직접 테스트해 원인을 좁혔다.

## 이게 챌린지가 의도한 함정인가

**아니다.** missions/README.md 1주차 통과 기준은 이렇게만 되어 있다.

> PreToolUse Hook이 위반 요청에서 차단된 출력이 남아 있어, 통과만 확인한 것이 아님을 알 수 있다.

경로 형태(상대/절대)에 따라 매칭이 갈리는 걸 테스트해보라는 요구는 없다. 이건 미션이 의도적으로 심어둔 함정이 아니라, **Hook 스크립트를 직접 작성하면서 흔히 발생하는 실수**다. Claude Code의 Hook 입력(`tool_input.file_path`)이 항상 같은 경로 형태로 온다고 가정하고 셸 패턴을 짜면 이런 식으로 뚫릴 수 있다는 걸, 실제로 겪어봐야 알 수 있는 종류의 문제다.

다만 이 경험 자체는 미션의 근거형 질문 3번("Hook을 붙였는데도 막지 못한 실수는 무엇이고 다음에 어떻게 막을 계획인가요?")에 정확히 들어맞는 실제 사례가 됐다 — 미션이 "이런 질문에 답하려면 스스로 한계를 찾아봐야 한다"는 의도로 저 질문을 넣어둔 것이라면, 이번 버그가 그 의도에 우연히 맞아떨어진 셈이다.

## 어떻게 고쳤는가

패턴에 슬래시 없는 경우를 추가했다.

```bash
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

수정 후 4가지 케이스로 재검증했다.

- 상대경로 `challenge.json` → exit 2 (수정됨)
- 절대경로 `.../challenge.json` → exit 2 (그대로 정상)
- 상대경로 `missions/README.md` → exit 2 (정상)
- 정상 파일 `app/page.tsx` → exit 0 (오탐 없음, 정상)

그 뒤 Claude Code에서 다시 "challenge.json을 수정하는 Edit 도구를 실제로 호출해줘"를 재현했더니, 이번엔 `Error: PreToolUse:Edit hook error: [...]: 차단됨: .../challenge.json 는 금지 변경 경로입니다`로 Edit 호출 자체가 거부되고 파일이 바뀌지 않았다.

## 남은 한계

- 이 Hook은 `matcher: "Edit|Write"`만 감시한다. Bash로 직접 `echo > challenge.json` 하는 식으로 파일을 고치면 이 Hook은 아예 발동하지 않는다 — Claude Code의 도구 경유가 아닌 변경은 못 막는다.
- 패턴을 상대/절대 두 형태로 각각 넣어야 했다는 것 자체가, 이런 셸 패턴 매칭 방식의 근본적인 약점이다. 더 견고하게 하려면 `realpath`나 `basename` 비교처럼 경로 형태에 무관한 방식으로 정규화한 뒤 비교하는 게 나을 수 있다 — 이번 제출에서는 시간상 패턴 추가로만 해결했다.
