# latency

**"하나의 작업이 시작부터 완료까지 걸리는 시간"**

단일 요청이 처리되는 데 얼마나 걸리는지를 측정하는 지표다. 사용자가 버튼을 눌렀을 때 응답이 돌아오기까지의 시간이 latency다.

---

## throughput과의 차이

latency는 개별 작업의 속도, throughput은 단위 시간당 처리량이다. 이 둘은 독립적으로 움직일 수 있다.

```
오토바이: latency 1시간, throughput 1명/시간
버스:     latency 2시간, throughput 25명/시간
```

버스는 오토바이보다 느리지만 처리량은 25배 높다. latency를 낮추는 것이 throughput을 높이는 것과 항상 같은 방향이 아니다.

---

## 동시성에서의 역할

동시성은 latency를 두 가지 방식으로 개선한다.

1. 줄인다 — 작업을 쪼개서 병렬 처리하면 완료 시간이 단축된다
2. 숨긴다 — I/O 대기처럼 어쩔 수 없는 latency 동안 다른 작업을 처리한다

latency를 숨기는 것은 응답성이 중요한 시스템에서 핵심 전략이다.

---

## 한 줄 요약

> latency = 작업 하나의 완료 시간. throughput과 독립적으로 움직인다.

참고: throughput.md, concurrency.md
