# Pod

Pod는 쿠버네티스에서 컨테이너를 배포하는 최소 단위다. 컨테이너 하나일 수도 있고, 여러 개일 수도 있다.

## Pod가 존재하는 이유

컨테이너는 namespace로 격리된 프로세스다. 그런데 어떤 컨테이너들은 항상 같이 다녀야 한다. 예를 들어 앱 컨테이너와 로그 수집 사이드카는 같은 파일을 봐야 하고, 같은 네트워크에 있어야 한다. Pod는 "이 컨테이너들은 항상 같이 배치되어야 해"를 표현하는 단위다.

## 내부 구조: namespace 공유

Pod 안의 컨테이너들은 일부 namespace를 공유한다.

- net namespace 공유 — 같은 IP를 쓴다. 컨테이너끼리 `localhost`로 통신 가능
- ipc namespace 공유 — 프로세스 간 통신 공유
- mnt namespace는 각자 — 파일시스템은 기본적으로 분리. volume을 마운트하면 공유 가능

```
Pod (net namespace 공유)
├── pause 컨테이너        ← net namespace를 소유하는 껍데기
├── 컨테이너 A (앱)       ← pause의 net namespace에 합류
└── 컨테이너 B (사이드카) ← pause의 net namespace에 합류
```

## pause 컨테이너

Pod가 생성될 때 제일 먼저 뜨는 껍데기 컨테이너다. 아무 일도 안 하고, net namespace만 소유한다. 앱 컨테이너 A, B는 이 pause 컨테이너의 namespace에 합류하는 방식으로 같은 네트워크를 쓴다.

앱 컨테이너가 죽었다 살아나도 IP가 바뀌지 않는 이유가 이것 때문이다. namespace를 소유한 pause가 살아있는 한 IP는 유지된다.

## Pod 생성 흐름

```
kubectl apply
└── kube-apiserver → kubelet (노드 에이전트)
    └── containerd (컨테이너 런타임)
        └── runc
            ├── pause 컨테이너 시작 — net namespace 생성 및 소유
            ├── 컨테이너 A 시작 — pause의 net namespace에 합류
            └── 컨테이너 B 시작 — pause의 net namespace에 합류
```

- kubelet — 노드에서 돌면서 "이 Pod 띄워라" 지시를 받아 실행
- containerd — 이미지 다운로드, 파일시스템 준비
- runc — `clone()` 시스템콜로 namespace를 만들고 프로세스를 시작. 실제 Linux 커널 기능이 여기서 쓰인다

결국 Pod 생성도 runc가 namespace/cgroup을 설정하고 프로세스를 fork하는 것이다. 쿠버네티스는 그 위에서 어느 노드에 어떻게 배치할지를 관리하는 오케스트레이션 레이어다.

참고: container.md
