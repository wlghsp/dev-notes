# OCI (Open Container Initiative)
참고: kubernetes-container-runtime.md, container-image.md

---

컨테이너 이미지 포맷과 런타임의 표준을 정의하는 업계 표준화 기구다. Docker의 성공 이후 여러 컨테이너 런타임이 등장하면서 상호 호환성을 위해 만들어졌다.

OCI가 정의한 두 가지 표준이 있다.
- OCI Image Format Specification — 컨테이너 이미지의 표준 포맷
- OCI Runtime Specification — 컨테이너 생성·설정·실행의 표준 인터페이스

## CRI (Container Runtime Interface)

Kubernetes가 다양한 컨테이너 런타임을 지원하기 위해 정의한 인터페이스다. Kubelet은 CRI를 통해 런타임에 컨테이너 실행을 지시한다. 어떤 런타임이든 CRI를 구현하면 Kubernetes에서 사용할 수 있다.

CRI 구현체 중 하나인 CRI-O는 OCI 호환 런타임(rkt, runC, Kata Containers 등)을 Kubernetes와 연결해주는 경량 런타임이다.

## Docker와의 관계

Docker는 OCI 표준의 주요 멤버이며, Docker가 만든 이미지는 OCI 표준을 따른다. Kubernetes는 초기에 Docker만 지원했지만 지금은 CRI를 통해 여러 런타임을 지원한다. containerd(Docker 내부에서 분리된 런타임)가 현재 가장 널리 쓰인다.
