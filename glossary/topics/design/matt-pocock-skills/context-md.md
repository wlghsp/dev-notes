# CONTEXT.md — 프로젝트 공용 언어

## 한 줄 요약

프로젝트의 핵심 용어를 정의하는 파일 하나. 에이전트와 나 사이의 언어 갭을 없앤다.

---

## 왜 필요한가

에이전트는 프로젝트 전문 용어를 모른다. 그래서 "materialization cascade"를 설명하려면
"lesson inside a section of a course being made real by getting a spot in the file system"
이라는 20단어짜리 설명을 매번 해야 한다.

CONTEXT.md가 있으면 "materialization cascade"라는 1단어로 끝난다.
매 세션마다 이 절약이 누적된다.

---

## 파일 구조

단일 컨텍스트 레포:
```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

여러 컨텍스트가 있는 레포:
```
/
├── CONTEXT-MAP.md           ← 각 컨텍스트 위치 안내
├── docs/adr/                ← 시스템 전체 결정
└── src/
    ├── ordering/
    │   ├── CONTEXT.md       ← ordering 컨텍스트 용어
    │   └── docs/adr/
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

파일은 필요할 때 만든다. 첫 번째 용어가 확정될 때까지 만들지 않는다.

---

## CONTEXT.md 작성 원칙

오직 용어집이다. 아래 세 가지를 절대 쓰지 않는다:
- 구현 세부사항
- 스크래치패드
- 스펙 내용

형식 (domain-modeling 스킬 기준):
```markdown
# [프로젝트명] 도메인 언어

## 핵심 용어

**Materialization**
lesson이 파일 시스템에 실제 경로를 부여받는 것. 아직 파일 시스템에 없는 lesson은 virtual 상태.
_피할 것_: "real", "published", "created"

**Cascade**
...
```

---

## ADR (Architecture Decision Record)

결정을 기록하는 파일. docs/adr/0001-이름.md 형식.

만드는 기준 — 3가지를 모두 만족할 때만:
1. 되돌리기 어렵다 (스키마 변경, 외부 API 선택 등)
2. 맥락 없이 보면 이상해 보인다 (왜 이렇게 했지? 소리 나는 결정)
3. 실제 트레이드오프가 있었다 (진짜 대안들이 있었고 그 중 하나를 골랐다)

하나라도 빠지면 ADR 만들지 않는다.
"지금은 시간이 없어서" 같은 임시적 이유는 ADR에 쓸 이유가 아니다.

ADR 형식:
```markdown
# ADR-0001: [결정 제목]

## 상태
Accepted

## 맥락
왜 이 결정이 필요했는가.

## 결정
무엇을 선택했는가.

## 결과
이 결정의 영향.
```

---

## domain-modeling 스킬과의 관계

/domain-modeling은 CONTEXT.md를 능동적으로 구축하는 스킬이다.
용어에 이의를 제기하고, 엣지케이스 시나리오로 스트레스 테스트하고, 결정이 확정되는 순간 인라인으로 업데이트한다.

배치 처리하지 않는다. 용어가 확정되는 그 순간 파일에 쓴다.

---

## 실제 사용 예시

/grill-with-docs로 새 기능 논의 중:
```
나: "그러니까 lesson이 활성화되면..."
에이전트: "활성화"가 CONTEXT.md의 "Materialization"과 같은 건가요, 다른 건가요?

나: 같은 거야, materialization이 맞는 단어네

에이전트: [CONTEXT.md 즉시 업데이트]
          앞으로 "활성화" 대신 "materialization"을 씁니다.
```

이 수렴이 일어나는 순간마다 이후 모든 세션이 더 효율적이 된다.
