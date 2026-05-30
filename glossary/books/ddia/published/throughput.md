# throughput
참고: DDIA Ch.1 — Reliable, Scalable, and Maintainable Applications

---

단위 시간당 처리할 수 있는 작업의 양. 배치 처리 시스템에서 주로 중요한 성능 지표다.

Hadoop 같은 배치 시스템에서는 초당 처리할 수 있는 레코드 수, 또는 특정 크기의 데이터셋을 처리하는 데 걸리는 총 시간으로 측정한다.

---

## response time과의 관계

throughput과 response time은 서로 다른 시스템 특성을 나타낸다.

- 온라인 시스템(웹 서버, API): response time이 더 중요하다. 사용자가 직접 기다리기 때문이다.
- 배치 시스템(Hadoop, Spark): throughput이 더 중요하다. 사용자는 결과가 나올 때까지 기다리지 않고, 몇 시간 후에 완료된 결과를 본다.

둘이 반드시 비례하지는 않는다. 처리량을 최대화하려고 작업을 병렬로 몰아넣으면 각 요청의 response time이 늘어나기도 한다. 큐에 작업이 쌓이면 head-of-line blocking이 발생한다.

---

## scalability와의 연결

throughput은 scalability를 논할 때 핵심 load parameter 중 하나다. 배치 시스템에서 데이터가 두 배로 늘었을 때 처리 시간이 두 배로 느려지면, 자원을 두 배로 늘려야 하는가, 아니면 더 효율적인 알고리즘이나 파티셔닝이 필요한가를 판단하는 기준이 된다.

이상적인 배치 시스템에서 실행 시간 = 데이터셋 크기 / throughput이다. 실제로는 데이터 쏠림(skew)과 가장 느린 작업을 기다려야 하는 문제로 더 오래 걸린다.
