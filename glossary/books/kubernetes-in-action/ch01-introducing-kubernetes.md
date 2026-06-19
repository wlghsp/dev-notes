# Ch 01 — Introducing Kubernetes (종합)

Kubernetes in Action 2nd Edition, Chapter 1.
키워드 파일: kubernetes.md, borg-omega.md, declarative-model.md, control-plane.md, worker-node.md

---

## 왜 Kubernetes가 필요해졌나

모놀리식 애플리케이션 시대에는 하나의 프로세스를 강력한 서버 하나에 올리면 됐다. 스케일이 필요하면 서버를 업그레이드(수직 확장)했다.

마이크로서비스로 분리되면서 배포해야 할 단위가 수십~수백 개로 늘었다. 각각 독립적인 배포 사이클을 가지고, 서로 다른 라이브러리 버전을 요구한다. 컨테이너가 격리 문제를 해결했지만, 수백 개의 컨테이너를 수동으로 관리하는 건 불가능하다. Kubernetes는 이 자동화 문제를 해결하기 위해 등장했다.

Google은 이미 2000년대부터 Borg, Omega라는 내부 시스템으로 이 문제를 풀고 있었다. 그 경험을 바탕으로 2014년 Kubernetes를 오픈소스로 공개했다.

## Kubernetes가 하는 일

Kubernetes는 "클러스터를 위한 운영체제"다. OS가 CPU/메모리를 추상화해서 프로세스에 제공하듯, Kubernetes는 서버 클러스터 전체를 추상화해서 하나의 배포 공간으로 만든다. 개발자는 어느 서버에 배포될지 신경 쓰지 않는다.

핵심은 선언형 모델이다. "인스턴스 3개가 항상 실행 중이어야 한다"고 기술하면, Kubernetes가 현재 상태와 비교해서 알아서 맞춰준다. 인스턴스가 죽으면 재시작하고, 노드가 장애나면 다른 노드로 이동시킨다.

## 클러스터 구조

Kubernetes 클러스터는 두 종류의 노드로 나뉜다.

**Control Plane (Master nodes) — 클러스터의 뇌**

클러스터 전체를 제어한다. 4개의 핵심 컴포넌트로 구성된다.

- API Server — 모든 통신의 단일 진입점. 개발자도, 다른 컴포넌트도 API Server를 통해서만 통신한다
- etcd — 클러스터 상태를 저장하는 분산 key-value 저장소. API Server만 직접 접근한다
- Scheduler — 새 인스턴스를 어느 Worker node에 배치할지 결정한다
- Controllers — desired state와 current state를 지속적으로 비교하고 조정한다

**Worker nodes — 앱이 실제로 실행되는 곳**

각 Worker node에는 3개의 컴포넌트가 돌고 있다.

- Kubelet — API Server와 통신하며 이 노드에 할당된 컨테이너를 관리한다
- Container Runtime — Kubelet의 지시를 받아 실제로 컨테이너를 실행한다 (containerd)
- Kube Proxy — 네트워크 트래픽을 인스턴스들에 분산한다

## 애플리케이션 배포 흐름

manifest(YAML)를 API Server에 제출하면 이런 순서로 진행된다.

```
1. API Server가 manifest를 etcd에 저장
2. Controller가 새 오브젝트를 감지하고 인스턴스 오브젝트 생성
3. Scheduler가 각 인스턴스에 Worker node 배정
4. Kubelet이 Container Runtime에 컨테이너 실행 지시
5. Kube Proxy가 Service에 대한 로드밸런서 설정
6. Kubelet과 Controllers가 지속적으로 상태를 감시
```
