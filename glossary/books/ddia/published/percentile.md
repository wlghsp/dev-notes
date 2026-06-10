# percentile
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

response time은 단일 숫자가 아니라 분포다. 평균(mean)은 이상치 몇 개에 크게 왜곡되기 때문에 "전형적인 사용자"를 잘 표현하지 못한다. percentile이 더 정직하다.

전체 요청을 응답 시간 순으로 정렬했을 때, 특정 비율 이하에 해당하는 최대 응답 시간을 뜻한다.

- p50 (median): 요청의 절반이 이 값 이하의 response time을 경험한다. "전형적인" 요청을 표현한다.
- p95: 100개 요청 중 95개는 이 값 이하, 5개는 이 값을 초과한다.
- p99: 100개 중 1개만 이 값을 초과한다.
- p999 (p99.9): 1000개 중 1개만 이 값을 초과한다.

p95~p999를 tail latency라 부른다. 응답이 가장 느린 요청을 경험하는 사용자는 보통 서비스를 가장 많이 사용하는 고가치 사용자인 경우가 많다. Amazon은 내부 서비스 간 호출의 SLO를 p999 기준으로 정의한다. 수백 개의 마이크로서비스가 체인으로 연결되는 구조에서 tail latency amplification을 막기 위해서다.

percentile 지표는 SLO/SLA의 기준선으로 쓰인다.

참고: tail-latency-amplification.md
참고: slo-sla.md
