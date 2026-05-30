# throughput

**"시스템이 단위 시간당 처리할 수 있는 작업의 수"**

개별 작업의 속도가 아니라 전체적인 처리 능력을 측정하는 지표다. 초당 처리하는 요청 수, 시간당 처리하는 트랜잭션 수가 throughput이다.

---

## latency와의 차이

throughput이 높다고 latency가 낮은 것이 아니다.

```
오토바이: latency 1시간, throughput 1명/시간
버스:     latency 2시간, throughput 25명/시간
```

버스는 오토바이보다 2배 느리지만 throughput은 25배 높다. 하나의 요청을 빠르게 처리하는 것과, 많은 요청을 처리하는 것은 다른 문제다.

---

## 동시성에서의 역할

동시성은 throughput을 높이는 핵심 수단이다. 단일 스레드로 처리하던 작업을 여러 스레드나 프로세스가 나눠 처리하면, 같은 시간 안에 더 많은 작업을 완료할 수 있다.

---

## 한 줄 요약

> throughput = 단위 시간당 처리량. latency와 독립적이며, 동시성으로 개선 가능하다.

참고: latency.md, concurrency.md
