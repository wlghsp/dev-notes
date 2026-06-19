# NIC (Network Interface Card)

NIC는 장치가 네트워크에 연결하기 위한 인터페이스다. 물리 하드웨어일 수도 있고, 커널이 소프트웨어로 만든 가상 인터페이스일 수도 있다.

## 물리 NIC

서버나 PC에 장착하는 하드웨어 카드다. 이더넷 포트에 케이블을 꽂으면 네트워크에 연결된다.

각 물리 NIC에는 공장에서 부여된 고유한 MAC 주소가 있다. 이 주소로 같은 네트워크 안에서 장치를 식별한다.

## 가상 NIC (소프트웨어 인터페이스)

커널은 물리 하드웨어 없이도 NIC처럼 동작하는 인터페이스를 만들 수 있다. OS 입장에서 물리 NIC와 가상 NIC는 동일하게 취급된다.

대표적인 가상 인터페이스들:

- `lo` (루프백) — 자기 자신과 통신하는 인터페이스. IP는 항상 127.0.0.1. 외부로 나가지 않는다.
- `veth` — 컨테이너용 가상 NIC 쌍. 한쪽에 넣은 패킷이 반드시 다른 쪽으로 나온다. 참고: bridge.md
- `docker0` — Docker가 만드는 브리지. NIC를 묶는 소프트웨어 스위치. 참고: bridge.md

컨테이너 안에서 `eth0`로 보이는 것도 가상 NIC다. veth pair의 한쪽 끝을 컨테이너 network namespace 안으로 옮겨서 NIC처럼 쓴다. 참고: container-creation.md

## ifconfig / ip link로 확인

`ifconfig`나 `ip link show`로 현재 시스템의 모든 NIC(물리+가상)를 볼 수 있다.

macOS `ifconfig` 출력에서:
- `en0` — WiFi NIC (물리)
- `lo0` — 루프백 (가상)
- `utun0~5` — VPN 터널 인터페이스 (가상)

Linux Docker VM 안에서:
- `docker0` — 브리지 (가상)
- `vethXXX` — 컨테이너 veth 호스트 끝 (가상)
