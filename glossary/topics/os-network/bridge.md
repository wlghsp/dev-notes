# bridge (브리지)

브리지는 여러 네트워크 인터페이스를 하나의 네트워크 세그먼트로 묶어주는 소프트웨어 스위치다.

물리 세계의 스위치 허브와 같은 역할이다. 여러 장치를 꽂으면 서로 통신할 수 있게 된다.

## 동작 원리

브리지는 패킷이 들어오면 MAC 주소를 보고 어느 포트로 내보낼지 결정한다.

처음 본 MAC 주소면 일단 모든 포트로 내보낸다(flooding). 응답이 오면 어느 포트에 어떤 MAC 주소가 있는지 기억한다(learning). 이후 같은 MAC 주소로 패킷이 오면 해당 포트로만 내보낸다(forwarding).

이 MAC 주소 테이블을 브리지 포워딩 테이블이라고 한다.

## Linux에서의 브리지

Linux 커널은 소프트웨어로 브리지를 만드는 기능을 기본 제공한다. `ip link add docker0 type bridge` 같은 명령으로 생성한다.

Docker는 이 Linux 브리지 기능을 써서 `docker0`를 만든다. 컨테이너가 생성될 때마다 veth pair의 호스트 쪽 끝을 docker0에 연결(attach)한다.

```
docker0 브리지
  ├── vethbb468be  ← 컨테이너 #1 veth 호스트 끝
  └── veth12d48c5  ← 컨테이너 #2 veth 호스트 끝
```

같은 docker0에 연결된 컨테이너끼리는 브리지를 통해 직접 통신할 수 있다.

## 브리지가 커버하는 범위

브리지는 같은 네트워크 세그먼트 안에서만 동작한다. 즉 컨테이너 ↔ 컨테이너 통신은 브리지가 담당하지만, 컨테이너 → 외부 인터넷은 브리지 역할 밖이다.

외부 통신은 iptables NAT가 컨테이너 IP를 호스트 IP로 변환해서 내보낸다. 브리지는 그 이전 단계까지만 관여한다.

## 확인 방법

```bash
# 브리지 목록과 연결된 인터페이스 확인
ip link show type bridge
bridge link show
```

참고: container-creation.md, veth pair 개념은 container-creation.md에 있다.
