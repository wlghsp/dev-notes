# serial-execution

**"하나의 작업이 완전히 끝난 후 다음 작업이 시작되는 실행 방식"**

가장 단순한 실행 모델이다. 프로그램은 위에서 아래로 한 줄씩 순서대로 실행된다. 동시에 여러 일을 하지 않는다.

---

## sequential computation과의 관계

serial execution은 실행 방식이고, sequential computation은 연산 간의 의존 관계다.

sequential computation은 이전 결과가 다음 연산의 입력이 되는 구조를 말한다. 이 경우 순서를 바꿀 수 없다.

serial execution이라도 각 작업이 독립적이면, 순서를 바꾸거나 병렬로 실행할 수 있다. sequential computation이면 serial로 실행할 수밖에 없다.

---

## 한계

단일 코어에서 한 번에 하나씩 처리하므로, 코어가 여러 개 있어도 활용하지 못한다. I/O 대기 중에도 CPU가 그냥 놀게 된다.

---

## 한 줄 요약

> serial execution = 하나씩 순서대로. 단순하지만 자원을 낭비한다.

참고: sequential-computation.md, parallel-execution.md, concurrency.md
