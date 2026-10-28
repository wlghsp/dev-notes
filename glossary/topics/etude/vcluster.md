# vcluster

## 한 줄 요약

진짜 k8s 클러스터 하나 위에, "완전히 독립된 것처럼 보이는 가짜 클러스터"를 여러 개 만들어주는 도구. Etude에서는 KLID CMP 같은 고급 k8s 실습을 실습자마다 서로 방해 없이 진행하게 해주려고 쓴다.

---

## 왜 필요했는가 — namespace 격리의 한계

Etude의 기존 k8s 세트(6, 12, 13, 14번)는 **namespace 격리** 방식이다. 클러스터 1개를 여러 실습자가 같이 쓰되, 각자 `quest-xxxxxxxx`라는 자기만의 구역(namespace) 안에서만 Pod/Deployment/Service를 만든다.

```
[진짜 클러스터 1개]
  ├── quest-aaaa (실습자 A의 구역)
  ├── quest-bbbb (실습자 B의 구역)
  └── quest-cccc (실습자 C의 구역)
```

이 방식은 대부분의 기초 실습(Pod 만들기, ConfigMap 쓰기 등)에서는 충분하다. 하지만 **namespace로는 못 가르는 것들**이 있다.

- `kubectl get nodes` — namespace와 무관하게 클러스터 전체의 노드가 다 같이 보인다. "내 노드만 보고 싶다"가 안 된다.
- `kubectl get ns --all-namespaces` — 다른 실습자의 namespace 이름이 그대로 노출된다.
- ClusterRole, CRD 같은 **클러스터 전체에 적용되는 리소스** — namespace 하나에 갇혀있는 개념이 아니라서, 애초에 "내 구역 안에서만" 만들 수가 없다.
- 여러 명이 동시에 똑같은 이름의 namespace(`cmp-gateway` 등)를 만들려고 하면 충돌한다.

KLID CMP(오케스트로 실제 배포 절차) 실습은 정확히 이런 것들을 다룬다 — RBAC 설정, CRD가 있는 Helm 차트 설치, 클러스터 레벨 리소스 조작. namespace 격리로는 이 실습 자체가 성립하지 않는다.

---

## vcluster가 하는 일

**"진짜 클러스터 안에, 진짜처럼 보이는 가짜 클러스터를 Pod 하나로 띄운다."**

```
[호스트 클러스터 (진짜, 서버에 1개만 있음)]
  └── vcluster-pool-a  ← 이것도 사실은 그냥 Pod 하나
       (하지만 이 안에 들어가면 완전히 독립된 클러스터처럼 보임)
```

vcluster 안에 들어간 실습자는:
- `kubectl get nodes`를 쳐도 **자기 vcluster의 가짜 노드만** 보인다.
- 다른 실습자의 vcluster는 존재 자체를 모른다.
- ClusterRole, CRD를 자유롭게 만들어도 **자기 vcluster 안에서만** 유효하다 — 다른 사람이나 호스트 클러스터에 전혀 영향을 안 준다.

즉 "가짜 클러스터"라고는 하지만, 실습자 입장에서는 진짜 클러스터를 통째로 혼자 쓰는 것과 사실상 똑같이 느껴진다. 그런데 실제로는 호스트 클러스터 위에 가벼운 Pod 하나로 떠 있을 뿐이라, 클러스터를 진짜로 여러 개 만드는 것보다 훨씬 적은 자원(vcluster 1개당 약 200MB RAM)으로 같은 효과를 낸다.

---

## Etude에서 실제로 어떻게 쓰이는가 — pool 방식

vcluster를 만드는 데 시간이 걸린다(로컬 실측 약 33초 — 클러스터 하나가 부팅하는 시간이라 어쩔 수 없다). 실습자가 세트를 열 때마다 33초를 기다리게 하면 경험이 나쁘므로, **미리 만들어두고 대기시키는 방식(pool)**을 쓴다.

```
[서버가 켜지면]
  → vcluster 2개를 미리 만들어서 "대기 상태"로 세워둠 (pool)

[실습자가 k8s-isolated 세트를 열면]
  → 대기 중이던 vcluster 하나를 즉시 배정 (기다릴 필요 없음)
  → 방금 꺼내 쓴 만큼, 뒤에서 조용히 새 vcluster를 하나 더 만들어서 pool을 다시 채움

[실습자가 세션을 끝내면]
  → 배정됐던 vcluster는 삭제 (재사용하지 않음)
```

**왜 재사용하지 않고 매번 삭제하는가**: vcluster 안에서는 실습자가 클러스터 레벨 설정(RBAC 등)까지 자유롭게 바꿀 수 있다. 다음 사람에게 그대로 넘겨주면 이전 실습자의 흔적이 남아있을 위험이 있어서, 매번 깨끗한 새 vcluster를 준다.

---

## `POOL_SIZE`가 왜 2인가

지금 서버는 24GB RAM인데, vcluster 1개가 약 200MB밖에 안 쓴다. `POOL_SIZE=2`는 "동시에 2명까지는 기다림 없이 즉시 입장 가능"하다는 뜻이고, 3번째 사람이 몰리면 그 자리에서 새로 만들어(33초 대기) 처리한다(fallback). 지금은 vcluster를 쓰는 실제 콘텐츠(KLID CMP)가 아직 없어서 이 값을 늘릴 필요가 없는 상태다.

---

## 아직 실제로 안 쓰이고 있다는 점

지금(2026-07) 서버에는 vcluster pool 배관(코드)만 준비되어 있고, **실제로 vcluster를 쓰는 퀘스트 세트는 DB에 하나도 없다.** `docker ps`로 봤을 때 vcluster Pod 2개가 조용히 대기하고 있는 게 정상이다 — KLID CMP 콘텐츠가 만들어지면 그때 실제로 쓰이기 시작한다.

---

## "지금 우리가 쓰는 게 vcluster야 k3d야?" — sandbox 타입별 정리

이 질문이 헷갈리는 이유: 어느 쪽이든 **물리적으로는 같은 k3d 클러스터(`etude`, 노드명 `k3d-etude-server-0`) 위에서 동작**한다. 차이는 그 위에서 격리를 어떻게 하느냐다.

| sandbox type | 격리 방식 | 실제 쓰는 세트 (2026-07 기준) | 핸들러 |
|---|---|---|---|
| `k8s` | **namespace 격리** — k3d 클러스터를 공유하고 퀘스트별로 `quest-xxxxxxxx` namespace만 따로 씀 | 세트 6(k8s 기초), 12(ConfigMap/Secret), 13(스토리지/네트워크), 14(Helm 기초) | `handleK8sTerminal` (`backend/src/services/terminal.ts`) |
| `k8s-isolated` | **vcluster 격리** — pool에서 배정받은 vcluster(가짜 클러스터) 안에서 완전 격리 | 아직 DB에 등록된 세트 없음 (KLID CMP 콘텐츠 준비 중) | `handleK8sIsolatedTerminal` (`backend/src/services/terminal.ts`) |

즉 지금 실습자들이 실제로 만나는 "k8s 기초" 같은 세트는 **k3d(namespace 격리)**를 쓰고 있고, vcluster는 아직 실전 투입 전이다. `01_sandbox.sql`에서 어떤 세트가 어느 sandbox_type을 쓰는지 직접 확인하려면:

```sql
SELECT id, title, sandbox_type FROM quest_set;
```

---

## 관련 문서

- [kubeconfig.md](kubeconfig.md) — vcluster도 접속하려면 kubeconfig가 필요하고, 이걸 세션마다 새로 만들어야 하는 이유
- [guide_phase10_klid_cmp.md](../guides/guide_phase10_klid_cmp.md) — vcluster pool 구현 가이드, 실제로 겪은 트러블슈팅(TLS SAN, Secret 폴링, NodePort 충돌 등)
- [spec_phase10_klid_cmp.md](../specs/spec_phase10_klid_cmp.md) — 왜 namespace 격리로 안 되는지에 대한 상세 근거
- [k8s_cluster_isolation.md](../research/k8s_cluster_isolation.md) — namespace 격리 vs k3d per-user vs vcluster 세 가지 방식 비교 검토
