# parallel-execution

**"여러 작업이 물리적으로 동시에 실행되는 것"**

멀티코어 CPU에서 각 코어가 서로 다른 작업을 동시에 처리하는 것이 전형적인 예다. concurrency와 달리, parallel execution은 실제로 같은 순간에 여러 작업이 실행된다.

---

## concurrency와의 차이

concurrency는 여러 작업을 다루는 능력이다. 단일 코어에서도 빠르게 전환하며 여러 작업을 번갈아 처리하면 concurrent하다.

parallel execution은 물리적으로 동시에 실행되는 것이다. 멀티코어가 있어야 가능하다.

```
concurrency: 여러 작업을 관리 (단일 코어에서도 가능)
parallelism: 여러 작업을 실제로 동시에 실행 (멀티코어 필요)
```

모든 parallel execution은 concurrent하지만, concurrent한 것이 모두 parallel execution은 아니다.

---

## 전제 조건

병렬 실행이 가능하려면 작업들이 독립적이어야 한다. sequential computation처럼 앞 결과에 의존하는 작업은 병렬화할 수 없다. Amdahl's law가 말하는 병렬화의 이론적 한계가 여기서 나온다.

---

## 한 줄 요약

> parallel execution = 물리적으로 동시에 실행. 멀티코어가 필요하고, 독립적인 작업에만 적용 가능하다.

참고: serial-execution.md, concurrency-vs-parallelism.md, amdahls-law.md
