# container-creation (컨테이너 생성 과정)
참고: container.md

---

`docker run` 한 줄이 실행될 때 내부에서 일어나는 일이다.

1. **이미지 레이어 마운트** — 이미지를 구성하는 읽기 전용 레이어들을 overlay filesystem으로 쌓는다. 맨 위에 쓰기 가능한 레이어를 하나 추가해서 컨테이너가 파일을 수정할 수 있게 한다.

2. **namespace 생성** — `clone()` 시스템 콜에 namespace 플래그를 지정해서 새 프로세스를 생성한다. 이 시점부터 그 프로세스는 격리된 PID, 네트워크, 파일시스템 등을 갖는다.

3. **cgroup 설정** — CPU/메모리 제한을 커널 cgroup에 등록한다. 컨테이너 프로세스가 지정된 자원 이상 사용하면 커널이 강제로 막는다.

4. **네트워크 연결** — 가상 네트워크 인터페이스(veth pair)를 만들어 한쪽은 컨테이너 namespace 안에, 다른 쪽은 호스트 브리지에 연결한다. 이 구조로 컨테이너가 외부와 통신할 수 있다.

5. **프로세스 실행** — 격리된 namespace 안에서 이미지에 지정된 명령어(entrypoint)를 실행한다. 이 프로세스가 컨테이너 안에서 PID 1이 된다.

## 실제 실행 주체: Docker → containerd → runc

`docker run`은 시작점일 뿐이고, 실제 작업은 세 계층으로 위임된다.

- **Docker daemon** — CLI 요청을 받아서 containerd에 위임한다.
- **containerd** — 컨테이너 생명주기(시작/중지/삭제), 이미지 다운로드, 스토리지 관리를 담당하는 데몬. Docker에서 분리된 독립 프로젝트이며, Kubernetes도 Docker 없이 containerd를 직접 쓴다.
- **runc** — namespace/cgroup 시스템 콜을 실제로 호출하는 가장 하위 실행기. OCI 표준을 구현한 CLI 도구다. 컨테이너 프로세스를 "실제로 켜는" 마지막 단계를 수행한다.

이렇게 분리된 이유는 Docker가 너무 모놀리식이었기 때문이다. containerd를 독립시키면 Kubernetes 같은 오케스트레이터가 Docker 없이도 컨테이너를 실행할 수 있다. 실제로 Kubernetes는 Docker를 걷어내고 containerd를 직접 쓰는 방향으로 전환했다.
