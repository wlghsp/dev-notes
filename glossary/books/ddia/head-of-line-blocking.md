# head-of-line blocking
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

앞에 있는 느린 요청 하나가 뒤의 빠른 요청들을 전부 대기시키는 현상이다.

## 왜 발생하는가

서버가 동시에 처리할 수 있는 요청 수에 한계가 있다. 처리 슬롯이 꽉 찬 상태에서 그 중 하나가 느린 작업(DB 쿼리, 외부 API 호출 등)을 기다리고 있으면, 큐에 쌓인 빠른 요청들은 슬롯이 날 때까지 전부 대기해야 한다.

실제 처리는 1ms면 끝나는 요청이 수백ms를 큐에서 기다리는 일이 생긴다.

## response time 측정에 미치는 영향

성능 테스트 시 응답을 기다리지 않고 요청을 계속 쏟아붓는 방식(open-loop)으로 테스트하면, 큐가 쌓이면서 측정값이 실제보다 나빠 보인다. 느린 요청이 큐를 막아 뒤따르는 빠른 요청들의 응답 시간까지 끌어올리기 때문이다.

앞 요청 응답을 받은 뒤 다음 요청을 보내는 방식(closed-loop)으로 테스트해야 실제 트래픽 패턴에 가까운 수치가 나온다.

## tail latency와의 관계

head-of-line blocking은 tail latency를 두껍게 만드는 원인 중 하나다. 느린 요청 하나가 뒤따르는 요청들의 응답 시간 분포 전체를 오른쪽으로 밀어버린다.
참고: tail-latency-amplification.md, latency-vs-response-time.md
