# latency vs response time
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

두 용어는 자주 혼용되지만 의미가 다르다.

- response time: 클라이언트가 요청을 보내고 응답을 받기까지의 전체 시간. 네트워크 지연, 큐 대기, 실제 처리 시간, 가비지 컬렉션 멈춤 등 모든 것을 포함한다. 사용자가 실제로 경험하는 시간이다.

- latency: 요청이 처리되기를 기다리는 시간. 서비스를 기다리는 잠재적 대기 시간. response time의 부분 집합에 가깝다.

실무에서 response time이 더 유용한 개념이다. 같은 요청을 반복해도 매번 다른 시간이 걸린다. 네트워크 패킷 손실로 인한 재전송, OS의 컨텍스트 스위치, 다른 프로세스로 인한 CPU 경합, 페이지 폴트, GC pause, TCP 패킷 손실 등 수많은 변수가 존재한다. response time은 단일 숫자가 아니라 분포로 생각해야 한다.

참고: percentile.md
참고: head-of-line-blocking.md
