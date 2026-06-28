# Matt Pocock Skills 활용 가이드

출처: github.com/mattpocock/skills
설치: `npx skills@latest add mattpocock/skills`

---

## 이 폴더 구성

overview.md — 레포 전체 개요, 4가지 문제와 스킬 카탈로그
grill-me.md — 만들기 전 끝장 인터뷰
tdd.md — 수직 슬라이스 테스트 주도 개발
diagnosing-bugs.md — 버그 진단 6단계 프로세스
codebase-design.md — 깊은 모듈 설계 어휘와 원칙
prototype.md — 버리는 코드로 설계 검증
context-md.md — 프로젝트 공용 언어 관리
improve-codebase-architecture.md — 코드베이스 설계 개선 정기 점검
handoff.md — 대화 맥락 이어받기 문서

---

## 바로 쓸 수 있는 것들 (우선순위 순)

1. /grill-me 또는 /grill-with-docs
   코드 짜기 전에 항상. 뭘 만들지 불분명하면 항상 여기서 시작한다.

2. diagnosing-bugs 흐름
   스킬 없이도 원칙만 적용 가능. 피드백 루프 먼저, 코드 읽기 나중.

3. /tdd
   테스트 전체를 먼저 쓰는 습관 교정. 수직 슬라이스로만.

4. /improve-codebase-architecture
   며칠에 한 번. AI로 빠르게 짤수록 더 자주.

5. /prototype
   "이게 맞나?" 싶은 상태 모델이나 UI 아이디어가 있을 때.

6. /handoff
   긴 작업 중간에 세션을 끊어야 할 때.

---

## 스킬 호출 타입

User-invoked: /grill-me, /grill-with-docs, /improve-codebase-architecture, /prototype, /handoff, /to-prd, /to-issues
→ 내가 직접 `/명령어`로 호출

Model-invoked: /diagnosing-bugs, /tdd, /domain-modeling, /codebase-design, /grilling
→ 에이전트가 상황에 맞게 자동 호출 (직접 호출도 가능)
