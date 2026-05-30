# scalability

**"자원을 추가했을 때 시스템 성능이 그에 비례해서 늘어나는 특성"**

더 많은 요청을 처리해야 할 때, 시스템이 자원 추가만으로 대응할 수 있으면 확장 가능하다고 한다. 방향에 따라 vertical scaling과 horizontal scaling으로 나뉜다.

---

## vertical scaling (scale up)

기존 서버의 사양을 올리는 것이다. CPU를 더 빠른 것으로 교체하거나, 메모리를 늘린다.

한계가 명확하다. 하드웨어에는 물리적 상한이 있고, 고사양 장비일수록 가격 대비 성능 향상폭이 줄어든다. Moore's law가 한계에 부딪힌 이후로는 수직 확장만으로는 성능 천장을 뚫기 어렵다.

---

## horizontal scaling (scale out)

서버를 여러 대로 늘려서 부하를 분산하는 것이다. 클라우드 환경에서 인스턴스를 추가하는 것이 전형적인 예다.

이론적으로는 자원을 계속 추가할 수 있어 확장성이 높다. 하지만 여러 머신이 함께 일하려면 동시성이 전제가 된다. 한 대의 컴퓨터로 부족할 때, 여러 대를 연결한 computing cluster가 하나의 처리 단위로 동작한다.

---

## 한 줄 요약

> scalability = 자원 추가 시 성능이 비례해서 늘어나는 능력. 수평 확장이 현실적이며, 동시성이 전제 조건이다.

참고: concurrency.md, horizontal-scaling.md, vertical-scaling.md
