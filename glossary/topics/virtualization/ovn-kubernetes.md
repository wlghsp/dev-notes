# OVN-Kubernetes

OVN-Kubernetes는 OCP(쿠버네티스) 클러스터의 기본 CNI(Container Network Interface) 플러그인이다. Pod 간 통신, 서비스 라우팅, NetworkPolicy(누가 누구와 통신 가능한지에 대한 규칙)를 담당한다.

## 무엇을 하는가

Pod가 생성될 때 그 Pod에 IP를 할당하고, 다른 노드에 있는 Pod와도 통신 가능하도록 오버레이 네트워크를 구성한다. Docker의 `docker0` 브리지가 한 호스트 안에서만 컨테이너를 묶어주는 것과 달리, OVN-Kubernetes는 여러 노드에 걸친 Pod들을 하나의 가상 네트워크로 묶는다. 참고: bridge.md, pod.md

## OVN과의 관계

이름 그대로 OVN(Open Virtual Network) 엔진을 그대로 가져다 쓴다. OpenStack의 Neutron이 OVN을 드라이버로 쓰는 것과 동일한 관계다 — OVN-Kubernetes는 OVN을 쿠버네티스 API(Pod, Service, NetworkPolicy)에 맞게 연결해주는 컨트롤러 계층이라고 보면 된다.

참고: networking-overview.md, neutron.md, ocp.md
