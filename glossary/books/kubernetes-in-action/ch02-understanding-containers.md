# Ch 2. Understanding containers — 종합 정리

키워드 파일: container-image.md, linux-namespace.md, cgroup.md, dockerfile.md, container-registry.md, oci.md

---

## 왜 컨테이너인가

마이크로서비스가 많아지면 각 서비스마다 VM을 띄우기엔 리소스 낭비가 크다. 컨테이너는 VM 없이 프로세스 수준에서 격리를 제공한다. 별도의 OS 없이 호스트 커널을 공유하므로 훨씬 가볍고 빠르게 시작된다.

## 컨테이너 = 격리된 프로세스

컨테이너는 특별한 실행 환경이 아니다. Linux 커널의 두 가지 기능으로 격리된 일반 프로세스다.

**Linux Namespace** — 프로세스가 볼 수 있는 자원의 범위를 제한한다. 파일시스템(mnt), 프로세스(pid), 네트워크(net), 호스트명(UTS) 등 자원 종류별로 별도의 네임스페이스를 가진다. 컨테이너 안에서 `ps aux`를 실행하면 자신의 프로세스만 보이는 이유다.

**cgroup** — 프로세스가 사용할 수 있는 자원의 양을 제한한다. CPU, 메모리, 네트워크 대역폭 등을 제어한다. 네임스페이스가 "무엇을 보느냐"라면 cgroup은 "얼마나 쓰느냐"다.

호스트에서 `ps aux`를 실행하면 컨테이너 안의 프로세스도 보인다. 단, PID가 다르다. 컨테이너는 자체 PID 네임스페이스를 가지므로 내부에서 PID 1인 프로세스가 호스트에서는 다른 번호다.

## VM과의 차이

VM은 하이퍼바이저가 하드웨어를 가상화하고 각 VM이 별도의 커널을 가진다. 앱의 syscall이 guest OS 커널 → 하이퍼바이저 → 물리 CPU로 전달된다. 컨테이너는 호스트 커널에 직접 syscall하므로 오버헤드가 없다.

격리 수준은 VM이 더 강하다. 컨테이너는 같은 커널을 공유하므로 커널 취약점을 통한 탈출이 이론적으로 가능하다. 보안이 중요한 환경에서는 Capabilities, seccomp, AppArmor/SELinux 등의 추가 보안 레이어를 적용한다.

> 📷 Figure 2.2 (책 p.43) — VM vs 컨테이너의 syscall 경로
> 📷 Figure 2.3 (책 p.44) — 베어메탈 / VM / 컨테이너 비교

## Docker와 이미지

Docker는 컨테이너를 쉽게 다루게 해주는 플랫폼이다. 격리 자체는 Linux 커널이 하고, Docker는 그 위에서 편의성을 제공한다.

이미지는 앱과 실행 환경 전체를 묶은 패키지다. Dockerfile을 작성하면 `docker build`로 이미지를 만들고, 레지스트리에 push해서 배포한다. 다른 머신에서 pull 후 `docker run`으로 실행하면 어디서든 동일한 환경이 된다.

이미지는 레이어로 구성된다. Dockerfile 지시어마다 레이어가 하나씩 생긴다. 레이어는 여러 이미지 간에 공유되어 스토리지와 전송량을 줄인다. Copy-on-Write(CoW)로 컨테이너들이 같은 레이어를 공유해도 서로 간섭하지 않는다.

> 📷 Figure 2.4 (책 p.47) — 이미지, 레지스트리, 컨테이너의 관계
> 📷 Figure 2.14 (책 p.62) — Dockerfile 빌드 과정

## 표준화: OCI와 CRI

Docker 이후 컨테이너 생태계가 커지면서 표준이 필요해졌다. OCI가 이미지 포맷과 런타임 인터페이스를 표준화했다. Kubernetes는 CRI(Container Runtime Interface)로 여러 런타임을 지원한다. containerd, CRI-O 등이 CRI 구현체다.
