# latency vs response time
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

두 용어는 자주 혼용되지만 의미가 다르다.

- response time: 클라이언트가 요청을 보내고 응답을 받기까지의 전체 시간. 네트워크 지연, 큐 대기, 실제 처리 시간, 가비지 컬렉션 멈춤 등 모든 것을 포함한다. 사용자가 실제로 경험하는 시간이다.

- latency: 요청이 처리되기를 기다리는 시간. 서비스를 기다리는 잠재적 대기 시간. response time의 부분 집합에 가깝다.

실무에서 response time이 더 유용한 개념이다. 같은 요청을 반복해도 매번 다른 시간이 걸린다. 네트워크 패킷 손실로 인한 재전송, OS의 컨텍스트 스위치, 다른 프로세스로 인한 CPU 경합, 페이지 폴트, GC pause, TCP 패킷 손실 등 수많은 변수가 존재한다. response time은 단일 숫자가 아니라 분포로 생각해야 한다.

---

## 왜 평균이 아닌 percentile인가

평균(mean) response time은 몇 개의 이상치에 크게 왜곡된다. 평균은 "전형적인 사용자"를 잘 표현하지 못한다.

percentile이 더 정직하다.

- p50 (median): 요청의 절반이 이 값 이하의 response time을 경험한다. "전형적인" 요청을 표현한다.
- p95: 100개 요청 중 95개는 이 값 이하, 5개는 이 값을 초과한다.
- p99: 100개 중 1개만 이 값을 초과한다.
- p999 (p99.9): 1000개 중 1개만 이 값을 초과한다.

p95~p999를 tail latency라 부른다. tail latency가 중요한 이유가 있다. 응답이 가장 느린 요청을 경험하는 사용자는 보통 가장 많은 데이터를 가진, 즉 서비스를 가장 많이 사용하는 고가치 사용자인 경우가 많다. Amazon은 내부 서비스 간 호출의 SLO를 p999 기준으로 정의한다. 수백 개의 마이크로서비스가 체인으로 연결되는 구조에서 tail latency amplification을 막기 위해서다.
참고: tail-latency-amplification.md

percentile 지표는 SLO/SLA의 기준선으로 쓰인다. 참고: slo-sla.md

---

## head-of-line blocking
참고: head-of-line-blocking.md

서버가 병렬로 소수의 요청만 처리할 수 있을 때, 느린 요청 하나가 뒤에 오는 빠른 요청들을 대기시킨다. 이를 head-of-line blocking이라 한다.

클라이언트 측에서 response time을 측정할 때는 이 현상을 고려해야 한다. 테스트를 위해 요청을 인위적으로 지연시키지 않고 계속 보내면 (open-loop), 큐가 쌓여 측정값이 실제보다 나빠 보인다.
