# tail latency amplification
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

여러 서비스를 순차 호출할 때, 하나라도 느린 케이스에 걸리면 전체 응답이 느려지는 현상이다.

## 왜 발생하는가

서비스 A가 B, C, D를 순서대로 호출한다고 하자.

```
사용자 → A → B → C → D → 응답
```

각 서비스의 p99가 10ms라는 것은 각각 1% 확률로 느린 케이스가 발생한다는 뜻이다.

B, C, D 중 하나라도 느릴 확률은 약 3%다. 즉 A의 최종 응답은 p97 수준에서 이미 tail에 걸린다. 호출 체인이 길어질수록 이 확률은 더 올라간다.

느린 케이스가 체인을 거치며 증폭(amplify)되기 때문에 amplification이라 부른다.

## 실무 영향

개별 서비스를 아무리 최적화해도, 체인 구조 자체가 tail을 두껍게 만든다. 서비스 수가 늘수록 전체 시스템의 tail latency는 나빠진다.

이 때문에 Amazon은 내부 서비스 간 호출의 SLO를 p99가 아닌 p999 기준으로 잡는다. 개별 서비스의 p999를 보장해야 체인 전체의 p99를 지킬 수 있기 때문이다.
참고: slo-sla.md, latency-vs-response-time.md
