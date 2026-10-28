# kubeconfig

## 한 줄 요약

`kubectl`이 "어느 k8s 클러스터에, 누구로, 어떻게 접속할지"를 적어둔 설정 파일. 이 파일이 없으면 `kubectl`은 어디에 말을 걸어야 할지 모른다.

---

## 왜 필요한가

`kubectl get nodes` 같은 명령을 실행하면, 내부적으로는 클러스터의 API 서버(웹 서버 같은 것)에 HTTP 요청을 보내는 것과 같다. 요청을 보내려면 최소한 이 세 가지가 필요하다.

1. **어디로** — API 서버 주소 (`server: https://...`)
2. **누구로** — 인증 정보 (클라이언트 인증서, 토큰 등)
3. **이 서버가 맞는지** — TLS 인증 기관(CA) 정보

kubeconfig는 이 세 가지를 YAML로 묶어놓은 파일이다. 보통 `~/.kube/config`에 있다.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTi...   # CA 인증서
    server: https://k3d-etude-server-0:6443          # API 서버 주소
  name: k3d-etude
contexts:
- context:
    cluster: k3d-etude
    user: admin@k3d-etude
  name: k3d-etude
current-context: k3d-etude
users:
- name: admin@k3d-etude
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...
```

---

## Etude에서 왜 이게 문제가 되는가

Etude의 백엔드(`backend`)는 컨테이너 안에서 돈다. 실습자가 k8s 퀘스트를 시작하면, 백엔드가 `etude-k8s` 컨테이너를 새로 만들고 그 안에서 `kubectl`을 쓸 수 있게 해줘야 한다.

문제는 이 `etude-k8s` 컨테이너가 **새로 뜨는 컨테이너**라, 기본적으로 kubeconfig 파일이 그 안에 없다는 것이다. `kubectl`을 실행해도 "설정이 없다"는 에러만 난다.

해결책은 **호스트(서버)에 있는 kubeconfig 파일을 컨테이너 안에 그대로 복사해 넣어주는 것** — 이게 바로 `binds`(볼륨 마운트)다.

```
["{KUBECONFIG_HOST_PATH}:/root/.kube/config:ro"]
     ↑ 호스트의 실제 경로            ↑ 컨테이너 안의 경로   ↑ 읽기 전용
```

이 한 줄이 하는 일: "호스트의 이 파일을, 컨테이너 안 `/root/.kube/config` 자리에 그대로 보이게 해줘. 단, 컨테이너 안에서는 수정 못 하게(`ro`, read-only) 해줘."

`kubectl`은 기본적으로 `~/.kube/config`(root 계정이면 `/root/.kube/config`)를 자동으로 찾아 읽으므로, 이 마운트만 되어 있으면 컨테이너 안에서 `kubectl get nodes`를 치는 순간 알아서 이 파일을 읽고 접속한다.

---

## `{KUBECONFIG_HOST_PATH}` — 왜 자리표시자로 되어 있는가

`sandbox` 테이블의 `binds` 컬럼에는 실제 경로 대신 `{KUBECONFIG_HOST_PATH}`라는 플레이스홀더가 들어있다.

```sql
('k8s', 'etude-k8s', '["{KUBECONFIG_HOST_PATH}:/root/.kube/config:ro"]', ...)
```

이유는 **호스트의 kubeconfig 경로가 로컬 개발 환경과 서버 배포 환경에서 다르기 때문**이다. DB에 절대경로를 하드코딩해버리면 로컬에서 테스트한 값이 서버에서는 틀린 경로가 된다.

그래서 실제 마운트 직전에, 코드에서 이 플레이스홀더를 그때그때 진짜 경로로 치환한다.

```typescript
// backend/src/services/sandbox.ts
export async function getSandboxConfig(sandboxType: string) {
    const [rows] = await db.query(...)
    const config = { image: row.image, binds: JSON.parse(row.binds), ... }

    if (config.binds) {
        const kubeconfig = process.env.KUBECONFIG_PATH ?? `${process.env.HOME}/.kube/config`
        config.binds = config.binds.map((b) =>
            b.replace('{KUBECONFIG_HOST_PATH}', kubeconfig)
        )
    }
    return config
}
```

`KUBECONFIG_PATH` 환경변수(`.env`/`.env.prod`)가 있으면 그 값을, 없으면 기본값(`$HOME/.kube/config`)을 쓴다. 즉:

- **로컬 개발**: `KUBECONFIG_PATH` 비워두면 `$HOME/.kube/config` (k3d가 로컬 클러스터를 만들 때 이 표준 위치에 등록해준다)
- **서버 배포**: `.env.prod`에 `KUBECONFIG_PATH=/root/.kube/config-etude`로 명시 (표준 위치가 아닌 별도 경로를 쓰는 이유는 아래 참고)

---

## 서버에서는 왜 `config-etude`라는 별도 파일을 쓰는가

[guide_phase8_deploy.md](../guides/guide_phase8_deploy.md) Step 4를 보면, k3d 클러스터를 만든 뒤 표준 kubeconfig(`~/.kube/config`)를 그대로 쓰지 않고 별도 파일을 하나 더 만든다.

```bash
k3d cluster create etude --api-port 127.0.0.1:6443

kubectl config view --raw | \
  sed 's|https://127.0.0.1:6443|https://k3d-etude-server-0:6443|g' \
  > ~/.kube/config-etude
```

**이유**: k3d가 기본으로 만드는 kubeconfig는 API 서버 주소를 `https://127.0.0.1:6443`(로컬호스트)로 적어둔다. 이건 "서버에서 직접 `kubectl`을 칠 때"는 맞는 주소지만, **Docker 컨테이너 안에서** 같은 주소로 접속하려 하면 실패한다 — 컨테이너 입장에서 `127.0.0.1`은 자기 자신을 가리키지, 호스트를 가리키지 않기 때문이다(컨테이너와 호스트는 별개의 네트워크 네임스페이스).

그래서 주소를 `k3d-etude-server-0`(k3d가 만든 클러스터 노드 컨테이너의 이름)으로 바꾼 별도 파일을 만든다. 같은 Docker 네트워크(`k3d-etude`) 안에 있는 컨테이너끼리는 컨테이너 이름으로 서로를 찾을 수 있기 때문에, 이 이름으로 바꾸면 `etude-k8s` 컨테이너 안에서도 정상적으로 클러스터에 닿는다.

이 원리는 Phase 10(vcluster)에서 `k3d-etude-server-0` 노드 이름을 그대로 재사용하는 이유이기도 하다 — [k8s_cluster_isolation.md](../research/k8s_cluster_isolation.md), [guide_phase10_klid_cmp.md](../guides/guide_phase10_klid_cmp.md) Step 2 참고.

---

## `k8s`(namespace 격리) vs `k8s-isolated`(vcluster) 에서 kubeconfig 다루는 방식 차이

Etude에는 kubeconfig를 다루는 방식이 다른 두 sandbox 타입이 있다.

| | `k8s` (Phase 6) | `k8s-isolated` (Phase 10) |
|---|---|---|
| kubeconfig | 서버 전체가 공유하는 **고정 파일 1개** (`~/.kube/config-etude`) | 세션(vcluster)마다 **매번 새로 생성**되는 파일 |
| DB `binds` 값 | `{KUBECONFIG_HOST_PATH}` 플레이스홀더 → `sandbox.ts`가 치환 | `NULL` — DB에 값 자체가 없음 |
| 실제 마운트 처리 | `sandbox.ts`의 `getSandboxConfig()` | `terminal.ts`의 `handleK8sIsolatedTerminal()`에서 개별 처리 |

`k8s-isolated`가 `binds`를 DB에 안 두는 이유는, 세션마다 완전히 다른 vcluster(다른 kubeconfig 파일)를 써야 해서 "고정된 하나의 경로"라는 전제 자체가 성립하지 않기 때문이다. 자세한 내용은 [guide_phase10_klid_cmp.md](../guides/guide_phase10_klid_cmp.md) Step 3, Step 6 참고.

---

## 관련 문서

- [sandbox_table.md](sandbox_table.md) — `sandbox` 테이블 전체 구조
- [guide_phase6.md](../guides/guide_phase6.md) — `k8s` 타입 최초 구현
- [guide_phase8_deploy.md](../guides/guide_phase8_deploy.md) Step 4 — 서버에서 `config-etude` 만드는 절차
- [guide_phase10_klid_cmp.md](../guides/guide_phase10_klid_cmp.md) — vcluster별 kubeconfig를 세션마다 만드는 방식
