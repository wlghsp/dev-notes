# 3의 법칙 (The Rule of Three)

참고: Refactoring 2nd Edition, Chapter 2 — When Should We Refactor? (Don Roberts가 파울러에게 준 지침)

## 규칙

> 처음 뭔가를 할 땐 그냥 한다. 비슷한 걸 두 번째로 할 땐, 중복이 마음에 걸리지만 일단 그대로 중복해서 한다. 비슷한 걸 세 번째로 할 때는, 리팩토링한다.

야구식으로 말하면: **삼진 아웃, 그러면 리팩토링한다(Three strikes, then you refactor).**

## 왜 즉시 추상화하지 않는가

첫 번째 등장에서 바로 공통 부분을 추상화하려 하면, 아직 패턴이 뭔지 정확히 모르는 상태에서 잘못된 추상화를 만들 위험이 크다. 두 번째 반복까지는 그 불편함을 감수하고 지켜본다. 세 번째 반복에 이르러서야 "정말 반복되는 패턴이 뭔지"가 뚜렷해지고, 그때 리팩토링(주로 Extract Function이나 Parameterize Function 같은 기법)으로 통합한다.

## preparatory-refactoring과의 관계

이 규칙은 "기능을 다 만들고 나서 청소하듯 리팩토링"하는 게 아니라, 코드를 짤 때 자연스럽게 리듬으로 스며드는 판단 기준이다. 비슷한 패턴이 세 번째 나타나는 시점이, 대개 preparatory-refactoring.md에서 말하는 "지금 구조를 바꿔두면 이번 기능이 더 쉬워지는" 순간과 겹친다.
