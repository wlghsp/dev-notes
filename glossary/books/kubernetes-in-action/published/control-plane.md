# control-plane (컨트롤 플레인)
참고: worker-node.md, declarative-model.md, kubernetes.md

---

Kubernetes 클러스터의 두뇌다. 클러스터 전체를 제어하는 컴포넌트들이 여기에 모여 있다. Master node들이 Control Plane을 구성한다.

## 구성 컴포넌트

**API Server**
클러스터의 단일 진입점. 개발자나 운영자가 클러스터를 조작할 때도, Kubernetes 내부 컴포넌트들이 상태를 읽고 쓸 때도 모두 API Server를 통한다. RESTful API를 제공한다. API Server 자체는 stateless다 — 상태는 etcd에 저장한다.
ㅈ
**etcd**
클러스터의 모든 상태를 저장하는 분산 key-value 저장소다. API Server만 etcd에 직접 접근한다. 다른 컴포넌트들은 API Server를 통해서만 상태를 읽고 쓴다.

**Scheduler**
새로 생성된 애플리케이션 인스턴스를 어느 Worker node에서 실행할지 결정한다. 각 노드의 가용 자원(CPU, 메모리)과 애플리케이션의 요구사항을 고려해서 배치한다.

**Controllers**
원하는 상태(desired state)와 현재 상태(current state)를 지속적으로 비교하고 맞춰주는 컴포넌트들이다. 예를 들어 Deployment Controller는 "인스턴스 3개"라고 선언했을 때 실제로 3개가 실행 중인지 감시하고, 부족하면 새로 만든다.

## 역할 요약

```
개발자/운영자
  → API Server (단일 진입점)
    → etcd (상태 저장)
    → Scheduler (어느 노드에?)
    → Controllers (원하는 상태 유지)
      → Worker nodes에서 실제 실행
```

Control Plane은 클러스터를 제어하지만, 애플리케이션을 직접 실행하지는 않는다. 실제 실행은 Worker node의 몫이다. 참고: worker-node.md
