# Container (컨테이너)

컨테이너는 새로운 OS나 커널을 띄우는 게 아니다. **Linux 커널에 원래 있던 기능**으로 프로세스를 격리하는 것이다.

## 핵심 원리: namespace와 cgroup

**namespace — 격리**

프로세스가 "보는 세계"를 제한한다. 같은 커널 위에 있어도, 각 컨테이너는 자기만의 파일시스템, 네트워크, 프로세스 목록을 가진 것처럼 동작한다.

namespace 종류:
- `mnt` — 파일시스템 마운트 포인트 격리
- `pid` — PID 격리. 컨테이너 안에서 PID 1이 따로 존재
- `net` — 네트워크 인터페이스 격리
- `uts` — 호스트명 격리
- `ipc` — 프로세스 간 통신 격리
- `user` — UID/GID 격리

**cgroup (control group) — 자원 제한**

CPU, 메모리, 디스크 I/O 등 자원 사용량을 컨테이너별로 제한한다. "이 컨테이너는 CPU 2코어, 메모리 512MB까지만" 같은 제약을 커널 수준에서 강제한다.

## 컨테이너를 띄운다는 것

새 커널을 올리는 게 아니라, **격리된 프로세스를 하나 시작하는 것**이다.

```
Linux 커널 (하나)
├── 컨테이너 A — namespace로 격리된 프로세스, cgroup으로 자원 제한
├── 컨테이너 B — namespace로 격리된 프로세스, cgroup으로 자원 제한
└── 컨테이너 C — namespace로 격리된 프로세스, cgroup으로 자원 제한
```

그래서 컨테이너는 VM보다 훨씬 가볍다. VM은 커널을 새로 부팅하지만, 컨테이너는 이미 돌고 있는 커널 위에서 프로세스 하나를 격리해서 시작할 뿐이다.

## Docker가 하는 일

Docker는 컨테이너 생성과 관리를 편하게 해주는 도구다. Linux 커널에 namespace/cgroup이 있어서 컨테이너를 만들 수 있지만, 이걸 직접 쓰려면 시스템콜을 직접 호출해야 한다. Docker는 그 위에서 `docker run`, `docker build` 같은 인터페이스를 제공한다.

- 이미지(Image) — 컨테이너가 시작할 때 쓸 파일시스템 스냅샷. 레이어 구조로 되어 있어 공통 레이어는 공유한다.
- 컨테이너 실행 — 이미지를 기반으로 namespace/cgroup을 설정하고 프로세스를 시작
- containerd / runc — Docker 내부에서 실제로 namespace/cgroup을 다루는 런타임

참고: container-creation.md

## Linux에서만 동작하는 이유

namespace와 cgroup은 Linux 커널 기능이다. 그래서 컨테이너는 Linux 커널 위에서만 네이티브로 동작한다.

macOS나 Windows에서 Docker를 쓸 수 있는 건, 내부적으로 Linux VM을 하나 띄우고 그 안에서 컨테이너를 돌리기 때문이다. 참고: hypervisor.md

참고: kernel.md
