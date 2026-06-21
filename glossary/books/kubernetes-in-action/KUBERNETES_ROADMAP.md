# Kubernetes in Action 2nd Edition — 학습 로드맵

책: Kubernetes in Action, Second Edition MEAP V15 — Marko Lukša (Manning, 2023)
파일: assets/kubernetes-in-action-2nd.pdf

---

## Part 1 — 기초 (Ch 1~4)

- [x] Ch 1. Introducing Kubernetes — Kubernetes가 무엇인지, 왜 등장했는지, 아키텍처 개요
  - ch01-introducing-kubernetes.md (종합)
  - kubernetes.md
  - borg-omega.md
  - declarative-model.md
  - control-plane.md
  - worker-node.md

- [x] Ch 2. Understanding containers — 컨테이너 원리, namespace/cgroup, 이미지
  - ch02-understanding-containers.md (종합)
  - container-image.md
  - linux-namespace.md
  - cgroup.md
  - dockerfile.md
  - container-registry.md
  - oci.md

- [ ] Ch 3. Deploying your first application — 첫 배포 실습

- [ ] Ch 4. Introducing Kubernetes API objects — API 오브젝트 개념, kubectl 기초

## Part 2 — 핵심 워크로드 (Ch 5~10)

- [ ] Ch 5. Running workloads in Pods — Pod 개념과 실행

- [ ] Ch 6. Managing the Pod lifecycle — Pod 생명주기

- [ ] Ch 7. Attaching storage volumes to Pods — 볼륨

- [ ] Ch 8. Persisting data in PersistentVolumes — PV/PVC

- [ ] Ch 9. Configuration via ConfigMaps, Secrets, and the Downward API

- [ ] Ch 10. Organizing objects using Namespaces and Labels

## Part 3 — 네트워크 & 운영 (Ch 11~17)

- [ ] Ch 11. Exposing Pods with Services — Service 개념

- [ ] Ch 12. Exposing Services with Ingress — Ingress

- [ ] Ch 13. Replicating Pods with ReplicaSets

- [ ] Ch 14. Managing Pods with Deployments

- [ ] Ch 15. Deploying stateful workloads with StatefulSets

- [ ] Ch 16. Deploying node agents and daemons with DaemonSets

- [ ] Ch 17. Running finite workloads with Jobs and CronJobs

---

## 완료 기준

각 챕터: 읽기 → 핵심 개념 glossary 정리
통합 이해: "Pod 하나가 뜨는 과정을 Control Plane부터 Worker Node까지 설명해봐"
