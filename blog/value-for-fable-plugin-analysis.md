# Value-for-Fable (VFF) 플러그인 분석

repo: https://github.com/itsinseong/value-for-fable

## 한 줄 요약

Claude Code용 플러그인. Sonnet에 "Fable 5(상위 모델)가 보이는 운영 패턴"을 프롬프트로 주입해서, Opus/Fable 대비 훨씬 싼 단가로 비슷한 응답 품질을 끌어내려는 시도. 모델을 바꾸는 게 아니라 output style + skill + agent + hook 조합으로 행동 패턴만 바꾼다.

## 문제의식

- Fable 5 $10/$50, Opus 4.8 $5/$25, Sonnet 4.6 $3/$15 (1M 토큰당 입력/출력).
- Sonnet은 싸지만 구조 없이 쓰면 품질 격차가 그대로 드러난다.
- 이 격차가 모델 능력 차이가 아니라 "운영 패턴(단서 우선 가설, 측정 먼저, 결론 먼저 말하기, 검증 후 완료 선언)" 차이에서 상당 부분 온다고 보고, 그 패턴을 Sonnet에 프롬프트로 이식.

## 구성 요소

레포는 마켓플레이스와 플러그인을 겸한다(`.claude-plugin/marketplace.json` + `plugin.json`).

```
value-for-fable/
├── skills/itsvff/SKILL.md      # 세션 모드 — 트리거로 수동 발동
├── agents/itsvff.md            # 위임 전용 서브에이전트 (2-pass 리뷰 백엔드)
├── output-styles/vff.md        # 상시 모드 v1 (원본 보존용)
├── output-styles/vff-v2.md     # 상시 모드 v2 (권장 — 압축 강제 제거판)
├── hooks/reminder.sh           # 장기 세션 드리프트 방지 훅
└── bench/                      # 재현 가능한 벤치 하네스 + 원자료
```

### skills/itsvff (세션 모드)
"VFF" / "패블 모드" 등 한국어 트리거로 발동. 발동 시 첫 줄에 "VFF 적용"을 강제 출력. 8개 섹션 규칙(communication, style_reference, effort_and_reasoning, tool_discipline, verification, code_and_changes, writing_and_research, tone_and_conduct, token_economy)으로 구성. 해당 세션에만 적용되고 다음 세션엔 재트리거 필요.

핵심 규칙 예:
- 첫 문장 = 결론, 산문 우선, 화살표 체인(A→B→실패)·토막문장 금지
- 완료 선언 전 검증 필수, 검증 못 한 건 "안 됐다"고 명시
- 진단 문제에서는 흔한 원인 나열보다 "모든 단서를 설명하는 가설"을 우선
- 되돌리기 쉬운 작업(읽기·분석)은 바로 행동, 파괴적 작업은 사전 확인

### agents/itsvff (위임용 서브에이전트)
사용자가 직접 부르지 않고 스킬이 내부적으로 위임. 고난도 과제나 "2-pass" 요청 시 초안을 별도 컨텍스트에서 리뷰. 리뷰 기준을 4개로 고정(요구사항 누락/사실 오류/미설명 단서/분량 초과)해서 자기비판의 노이즈를 줄임. 지식 격차가 의심되면 리뷰어를 Opus로 오버라이드 가능.

### output-styles (상시 모드)
`/config`로 설정하면 트리거 없이 모든 세션에 자동 적용. v1은 압축(토큰 절약)을 강하게 강제했는데, 벤치 결과 이게 품질을 오히려 깎아서 v2에서 압축 강제를 제거하고 진단/검증 구조만 남김.

### hooks/reminder.sh (드리프트 방지)
긴 세션에서 초반에 주입된 스킬 규칙이 컨텍스트 뒤로 밀려 흐려지는 걸 막는 장치. transcript가 400KB를 넘고 VFF가 활성 상태일 때만 매 턴 짧은 리마인더를 주입. 두 조건 중 하나라도 안 맞으면 아무것도 안 하고 종료(추가 비용 0).

## 벤치마크 결과 (bench/RESULTS.md 요약)

작성자가 자체적으로 6단계 실험을 거쳐 검증한 내용:

1. **v1의 압축 강제는 품질 부채였다.** ablation 테스트에서 압축을 뺀 버전이 압축 있는 v1보다 순위가 확실히 높았음. 분량 지정 과제에서 압축 강제가 글자 수 미달로 감점되는 사례도 확인.
2. **v2 ≈ Opus, 노이즈 안에서 동률.** 중립 채점표 + 독립 심판 2명으로 두 차례 재채점한 결과 v2가 Opus의 약 95~101% 수준. 단, 기본 진단·분량 맞추기 과제에서는 v2가 Opus와 같거나 앞서고, 깊은 추론(아키텍처 결정, 복잡 성능 진단)에서는 Opus가 5~7점 앞섬.
3. **3개 모델 가족 교차검증.** "Claude가 Claude를 채점해서 점수가 부풀었다"는 비판에 대응하기 위해 Gemini, GPT로도 같은 답변 쌍을 페어와이즈 채점. 세 가족 어디서도 Opus가 v2를 깨끗이 이긴 과제는 0건.
4. **VFF를 Opus에 씌우면 효과가 거의 없다.** 직접 측정한 결과 Opus+v2 vs 맨 Opus는 사실상 구분 불가. 즉 VFF의 효과는 Sonnet처럼 "덜 끌어내 쓰던 능력이 있는 모델"에서만 발생하고, 이미 강한 모델에는 마진이 작음.
5. 초기에 발표됐던 "5/6승·257점" 벤치는 원자료가 없어 재현 실패로 폐기, 이후 전부 원자료 공개하며 재검증.
6. v2 이후 시도한 v3(복합 진단 분해 추가)는 중립 기준으로 효과 없어 폐기.

전체적으로 방법론이 투명하다 — 실패한 벤치(v3, 원래 5/6승 주장)도 숨기지 않고 "반증됨"이라고 명시하고, 한계(표본 작음, 심판 대부분 Claude 계열, 진단/조언 중심 과제로 범위 한정)도 README와 RESULTS.md에 반복해서 명시.

## 핵심 주장과 한계

- **주장**: 이건 Opus의 추론 능력을 복제하는 게 아니라(distillation의 영역), Sonnet이 이미 갖고 있지만 프롬프트 구조 없이는 덜 쓰던 능력을 끌어내는 것(elicitation)이다. Chain-of-thought 프롬프팅이 같은 모델의 점수를 올리는 것과 같은 원리.
- **한계**: 순수 추론 천장이 필요한 과제(낯선 도메인 겹친 진단, 복합 아키텍처 결정)는 여전히 Opus가 앞선다. VFF는 이 천장 자체를 넘지 못한다.
- **출처 문제**: 8섹션 구조는 `elder-plinius/CL4R1T4S`에 공개된 Fable 5 시스템 프롬프트 관찰 내용을 "재구성"한 것이라고 명시. 원문 블록을 그대로 복제하지 않았다고 주장하지만, 이 출처 자체가 유출된 시스템 프롬프트 모음이라는 점은 유의할 부분.

## 설치 방법 (참고용)

```
/plugin marketplace add https://github.com/itsinseong/value-for-fable
/plugin install value-for-fable@itsinseong
```

약칭(`itsinseong/...`)으로 추가하면 SSH clone을 시도해 SSH 키 없는 환경에서 실패하므로 `https://` 풀 URL 사용 권장.
</content>
