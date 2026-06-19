# worker-node (워커 노드)
참고: control-plane.md, kubernetes-container-runtime.md

---

실제 애플리케이션(컨테이너)이 실행되는 서버다. Workload Plane을 구성한다. 클러스터는 수십~수백 개의 Worker node로 이뤄질 수 있다.

## 구성 컴포넌트

**Kubelet**
Worker node의 에이전트다. API Server와 통신하면서 이 노드에 할당된 컨테이너를 실행하고 상태를 보고한다. Container Runtime에 컨테이너 실행을 지시하는 주체이기도 하다.

**Container Runtime**
실제로 컨테이너를 실행하는 프로그램이다. Kubelet의 지시를 받아 namespace/cgroup을 설정하고 프로세스를 띄운다. containerd가 대표적이며, CRI(Container Runtime Interface)를 구현한 런타임이면 무엇이든 쓸 수 있다. 참고: kubernetes-container-runtime.md

**Kube Proxy**
네트워크 트래픽을 애플리케이션 인스턴스들에 분산한다. 각 Service에 대한 로드밸런서 역할을 한다. 이름은 proxy지만 실제로 트래픽이 통과하는 구조는 아니다 — iptables/ipvs 규칙을 설정하는 방식으로 동작한다.

## Control Plane과의 관계

Worker node의 컴포넌트들은 모두 API Server와만 통신한다. Worker node끼리 직접 통신하지 않는다.

```
Control Plane (API Server)
  ↕
Worker node
  - Kubelet → Container Runtime → 컨테이너 실행
  - Kube Proxy → 네트워크 트래픽 분산
```
