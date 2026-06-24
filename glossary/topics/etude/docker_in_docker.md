# Docker-in-Docker (DinD)

## 한 줄 요약

컨테이너 안에서 Docker 엔진을 실행해, 그 안에서 또 컨테이너를 만들 수 있는 구조.

---

## 왜 필요한가

Docker 퀘스트에서 `docker run`, `docker ps` 같은 명령어를 실습하려면 Docker 엔진이 필요하다.
호스트의 Docker 소켓을 마운트하는 방법도 있지만, 그러면 실습 컨테이너가 호스트 Docker를 직접 건드리게 된다.

DinD는 컨테이너 안에 독립된 Docker 데몬(`dockerd`)을 띄워 호스트와 완전히 격리된 환경을 만든다.

| | 호스트 소켓 마운트 | DinD |
|---|---|---|
| 격리 | X — 호스트 컨테이너가 보임 | O — 완전히 독립된 환경 |
| 권한 | 호스트 Docker 조작 가능 | 샌드박스 안에서만 |
| 실습 적합성 | 위험 | 안전 |

---

## 구조

```
호스트
└── docker:dind 컨테이너 (Privileged)
    └── dockerd 실행 중
        └── 실습자가 만드는 컨테이너들 (hello-world, nginx, ...)
```

---

## 핵심 설정

`docker:dind` 이미지는 Docker Hub 공식 이미지. 별도 빌드 불필요.
`Privileged: true` 없으면 dockerd가 뜨지 않는다 — 컨테이너 안에서 커널 기능을 써야 하기 때문.

```typescript
await docker.createContainer({
    Image: 'docker:dind',
    HostConfig: {
        Privileged: true,   // 필수
    },
})
```

---

## attach vs exec — DinD에서 왜 exec인가

일반 linux 컨테이너는 `container.attach()`로 bash 스트림을 바로 연결한다.
DinD는 컨테이너 시작 시 `dockerd`가 foreground로 실행되어 attach 스트림을 점유한다.
그래서 shell을 별도로 `exec`으로 붙여야 한다.

```
일반: createContainer(bash) → attach → 스트림 연결
DinD: createContainer(dockerd) → waitForDocker → exec(sh) → 스트림 연결
```

---

## waitForDocker

dockerd가 완전히 뜨기 전에 exec을 실행하면 실패한다.
소켓 파일(`/var/run/docker.sock`)이 생길 때까지 컨테이너 안에서 대기한다.

```typescript
async function waitForDocker(container) {
    const exec = await container.exec({
        Cmd: ['sh', '-c', 'until test -S /var/run/docker.sock; do sleep 0.2; done'],
    })
    await exec.start({})
    // exec이 종료될 때까지 폴링
}
```

소켓 파일이 생기면 sh가 루프를 탈출하고 종료 → exec.Running이 false → 다음 단계 진행.
