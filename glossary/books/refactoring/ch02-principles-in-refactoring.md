# Chapter 2 종합 정리

이 챕터에서 생성된 키워드 파일: refactoring-definition.md, two-hats.md, rule-of-three.md, preparatory-refactoring.md, comprehension-refactoring.md, litter-pickup-refactoring.md, planned-vs-opportunistic-refactoring.md, when-not-to-refactor.md, problems-with-refactoring.md, design-stamina-hypothesis.md, refactoring-and-the-wider-process.md, refactoring-and-performance.md

## 이 챕터의 성격

1장이 예제 하나로 리팩토링의 감을 잡는 챕터였다면, 2장은 코드 예시 없이 원칙과 판단 기준을 다루는 챕터다. 그래서 이 챕터는 각 원칙을 독립된 짧은 키워드 파일로 쪼개 정리했다 — 세부 내용은 각 파일 참조.

## 흐름 요약

**정의부터 시작** — refactoring-definition.md에서 리팩토링을 "관찰 가능한 동작은 바꾸지 않으면서 이해하기 쉽고 수정하기 저렴하게 만드는 것"으로 못박는다. two-hats.md는 이 작업을 기능 추가와 분리된 모드로 다루라는 실천 방법이다.

**언제 하는가** — rule-of-three.md(비슷한 게 세 번째 나타나면 리팩토링), preparatory-refactoring.md(기능 추가 직전이 가장 좋은 시점), comprehension-refactoring.md와 litter-pickup-refactoring.md(코드를 이해하는 과정 자체가 리팩토링의 계기가 되는 두 가지 패턴)까지가 "왜/언제 리팩토링하는가"에 대한 답이다. planned-vs-opportunistic-refactoring.md는 이 모든 게 대부분 계획 없이 자연스럽게 일어나야 한다는 걸 강조하고, when-not-to-refactor.md는 반대로 안 해도 되는 경우를 짚는다.

**부딪히는 현실** — problems-with-refactoring.md 하나에 코드 오너십, 브랜치/CI, 테스팅, 레거시 코드, 데이터베이스까지 다섯 가지 장애물을 모았다. 특히 데이터베이스 필드 리네이밍 사례에서 parallel-change.md(Change Function Declaration의 마이그레이션 절차)와 정확히 같은 구조가 재등장한다.

**더 큰 그림** — design-stamina-hypothesis.md(좋은 설계가 장기적으로 기능 추가 속도를 앞선다는 가설이자 YAGNI의 근거), refactoring-and-the-wider-process.md(자가 검증 코드·CI·리팩토링의 삼각 시너지), refactoring-and-performance.md(리팩토링과 성능이 방법론은 닮았지만 목적은 다르다는 구분, Ron Jeffries의 "측정 없이 추측하지 말라" 일화)로 챕터를 마무리한다.

## 한 줄 결론

리팩토링의 근거는 도덕("클린 코드")이 아니라 경제("더 빨리, 더 안전하게 바꿀 수 있다")다. 이 챕터의 모든 원칙은 결국 이 한 문장으로 수렴한다.
