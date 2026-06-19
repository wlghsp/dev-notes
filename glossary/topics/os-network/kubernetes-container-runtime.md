# kubernetes-container-runtime
참고: container-creation.md

---

Kubernetes는 컨테이너를 직접 실행하지 않는다. 컨테이너 런타임에 위임한다. 어떤 런타임을 쓸지는 CRI(Container Runtime Interface)라는 표준 인터페이스로 결정한다.

## CRI (Container Runtime Interface)

Kubernetes가 런타임과 통신하는 표준 인터페이스다. Kubernetes는 CRI만 구현하면 어떤 런타임이든 쓸 수 있다. containerd, CRI-O 등이 CRI를 구현한 런타임이다.

## 왜 Docker를 걷어냈나

초기 Kubernetes는 Docker만 지원했다. 그런데 Docker는 CRI를 구현하지 않았다. 그래서 Kubernetes가 Dockershim이라는 중간 변환 레이어를 직접 만들어서 Docker와 통신했다.

흐름이 이랬다.

```
Kubernetes → Dockershim → Docker daemon → containerd → runc
```

문제는 Dockershim을 Kubernetes 팀이 직접 유지보수해야 했다는 점이다. Docker가 내부 구조를 바꿀 때마다 Kubernetes도 따라서 수정해야 했다. 관리 부담이 컸다.

반면 containerd는 CRI를 직접 구현했다. 그래서 중간 레이어 없이 바로 연결된다.

```
Kubernetes → containerd → runc
```

Kubernetes 1.20에서 Dockershim deprecation을 공지했고, 1.24(2022년)에서 완전히 제거했다. Docker로 빌드한 이미지는 OCI 표준을 따르기 때문에 그대로 쓸 수 있다. 바뀐 건 런타임뿐이다.

## 정리

Docker는 개발자가 이미지를 빌드하고 로컬에서 실행하는 도구로 여전히 유효하다. 다만 Kubernetes 클러스터 안에서 컨테이너를 실행하는 런타임 역할은 containerd가 담당한다.
