# Week 3 리뷰 분석 및 다음 주차 보완 사항

PR #3 (submit/week-03__weekly-pr) 자동 리뷰 결과: 92/100, 통과.

## 잘한 점 (계속 유지할 것)

- 연관관계 주인/비주인, 편의 메서드 유무, LAZY 초기화 시점, cascade REMOVE vs orphanRemoval을 각각 대칭 테스트로 검증해 SQL 로그 기반 근거를 남긴 것
- 근거형 질문 4개 모두 테스트 클래스명과 실제 관찰 결과(1+3=4 쿼리, DB 정상/메모리 불일치 등)를 구체적으로 인용한 것
- 선택 근거에서 `CascadeType.ALL` 대신 `{PERSIST, REMOVE}`로 최소 선택한 이유와, Hibernate 버전별(6.2.13/6.4.10/6.5.3) 재현 실험 과정을 기록해 재현 가능하게 남긴 것

## 겪은 문제와 해결 (이번 주차 특이사항)

### self-check 실패 (0/100) → merge로 해결

- 문제: 첫 제출에서 `dingco-self-check`의 `submission-shape`가 실패해 0/100 처리됨
- 원인: `main`이 `.github/workflows/dingco-self-check.yml`을 업데이트했는데, 제출 브랜치가 그 이전 시점에서 갈라져 있어서 diff에 이 워크플로 변경이 "우리가 바꾼 것"처럼 잡힘 — "제출 PR에서는 채점 Workflow, challenge.json과 미션 원문을 바꿀 수 없습니다" 에러
- 해결: `git merge origin/main`으로 워크플로 파일을 최신화해서 push. `submission-shape`, `java-tests` 모두 pass로 전환
- 같은 문제를 challenge-algorithm-thinking 레포의 최신 PR에서도 동일하게 겪었음 — 두 레포 모두 관리자 측 워크플로 업데이트로 인한 것이라 코드 문제는 아니었음

## 보완할 점 (다음 주차부터 적용)

### 1. "리뷰 반영" 섹션이 여전히 비어있음 (week1과 같은 지적 반복)

- 문제: self-check 실패 → 재제출로 통과한 과정이 있었는데도, evidence의 "## 리뷰 반영"이 "자동 리뷰 수신 전"으로만 남아 있어 이전 지적(self-check 실패)에 대한 반영 기록이 명시적으로 없다는 지적을 받음
- 원인: week1 review-notes.md에서 이미 "PR 생성 후 리뷰 수신 시 반드시 반영 섹션을 채운다"는 체크리스트를 만들어뒀는데, 이번 주차 실행 과정에서 다시 놓침
- 다음 주차 적용: self-check 실패든 자동 리뷰 지적이든, **PR에 어떤 형태로든 피드백이 달리면 그 즉시 evidence의 "## 리뷰 반영"에 무엇을 어떻게 고쳤는지 기록**. week1 체크리스트를 실행 가이드에 실제로 넣어서 매 주차 마지막 단계로 강제할 것

### 2. 첨부한 패치(diff)에 일부 파일이 누락되어 리뷰어가 코드를 직접 못 봄

- 문제: `LazyProxyAndNPlusOneTest.java`, `CascadeAndOrphanRemovalTest.java`의 일부 패치가 생략되어, 리뷰가 evidence 요약만으로 판단함
- 원인: 정확한 원인 미확인(PR diff 표시 방식 또는 리뷰 시스템의 diff 읽기 범위 제한으로 추정)
- 다음 주차 적용: PR 생성/업데이트 후 실제로 GitHub PR의 "Files changed" 탭에서 새로 추가한 테스트 파일들이 전부 정상적으로 diff에 잡히는지 눈으로 직접 확인. 너무 크거나 이진 파일에 가까운 변경은 아닌지도 점검

## 체크리스트 (다음 주차 실행 가이드에 추가할 항목)

- [ ] PR 생성/업데이트 후 자동 리뷰(또는 self-check) 수신 → evidence "## 리뷰 반영"에 결과 기록 (지적 없으면 "지적 없음", 있으면 무엇을 어떻게 고쳤는지)
- [ ] self-check가 실패하면 원인을 로그로 직접 확인(`gh run view <run-id> --job <job-id> --log`) — "채점 Workflow/challenge.json 변경 금지" 에러라면 `git merge origin/main`으로 워크플로 최신화 후 재push
- [ ] PR의 "Files changed"에서 새로 추가한 파일들이 전부 정상적으로 diff에 표시되는지 확인
