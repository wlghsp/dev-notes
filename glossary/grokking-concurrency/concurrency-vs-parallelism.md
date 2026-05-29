# concurrency-vs-parallelism

**"동시성은 여러 작업을 다루는 능력, 병렬성은 여러 작업을 실제로 동시에 실행하는 것"**

둘은 자주 혼용되지만 다른 개념이다.

---

## 핵심 차이

concurrency는 구조의 문제다. 프로그램이 여러 작업을 동시에 진행 중인 상태를 관리할 수 있는가. 단일 코어에서도 빠르게 컨텍스트를 전환하면 concurrent하다.

parallelism은 실행의 문제다. 여러 작업이 물리적으로 같은 순간에 실행되는가. 멀티코어 없이는 불가능하다.

```
단일 코어, 컨텍스트 전환: concurrent O, parallel X
멀티코어, 각 코어가 다른 작업: concurrent O, parallel O
```

---

## 관계

모든 parallel execution은 concurrent하다. 하지만 concurrent하다고 해서 반드시 parallel execution은 아니다.

concurrent 프로그램을 작성하는 것이 먼저다. 그 위에서 하드웨어가 허용하면 parallel execution이 일어난다.

---

## 한 줄 요약

> concurrency = 여러 작업을 다루는 설계, parallelism = 물리적 동시 실행. concurrency가 전제, parallelism은 그 위에 올라온다.

참고: concurrency.md, parallel-execution.md, serial-execution.md
