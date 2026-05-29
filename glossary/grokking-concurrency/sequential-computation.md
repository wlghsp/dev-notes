# sequential-computation

**"이전 연산의 결과가 다음 연산의 입력이 되는 의존 관계"**

연산들 사이에 순서 의존성이 있는 구조다. A가 끝나야 B를 시작할 수 있고, B가 끝나야 C를 시작할 수 있다. 이 의존성이 있는 한 병렬화가 불가능하다.

---

## serial execution과의 차이

serial execution은 실행 방식(한 번에 하나씩)이고, sequential computation은 연산 간 의존 관계다.

작업들이 독립적이라면 serial execution이어도 순서를 바꾸거나 병렬화할 수 있다. sequential computation은 의존성 때문에 순서를 바꿀 수 없다.

```
독립적인 작업: A, B, C를 어떤 순서로든 실행 가능 → 병렬화 가능
sequential:   A → B → C 순서 고정 → 병렬화 불가
```

---

## 병렬화 가능 여부의 판단 기준

작업을 병렬화하려면 먼저 의존성 분석이 필요하다. sequential computation 구간은 병렬화할 수 없고, 이 구간이 전체 성능의 병목이 된다. Amdahl's law가 말하는 "직렬 부분"이 바로 이것이다.

---

## 한 줄 요약

> sequential computation = 이전 결과에 의존하는 연산 순서. 이 의존성이 병렬화의 한계를 만든다.

참고: serial-execution.md, parallel-execution.md, amdahls-law.md
