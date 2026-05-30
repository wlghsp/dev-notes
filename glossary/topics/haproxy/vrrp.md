# VRRP (Virtual Router Redundancy Protocol)

여러 라우터(또는 서버)를 묶어 하나의 가상 라우터처럼 보이게 하는 네트워크 프로토콜. RFC 5798로 표준화됐다.

HAProxy 이중화에서 Keepalived가 VRRP를 사용해 VIP 소유권을 관리한다.

## 핵심 개념

VRRP 그룹 안의 노드들은 하나의 `virtual_router_id`를 공유한다. 이 그룹 안에서 가장 priority가 높은 노드가 Master가 되고 VIP를 소유한다. 나머지는 Backup 상태로 대기한다.

Master는 주기적으로 VRRP advertisement(heartbeat)를 멀티캐스트로 보낸다. Backup들이 일정 시간 advertisement를 받지 못하면 Master가 죽었다고 판단하고, priority가 가장 높은 Backup이 새 Master가 된다.

## VIP 이전 과정

Master가 죽으면:
1. Backup이 Master로 전환한다.
2. 새 Master가 Gratuitous ARP를 브로드캐스트한다 — "VIP는 이제 내 MAC 주소로 오세요".
3. 같은 네트워크의 장비들이 ARP 캐시를 업데이트한다.
4. 이후 VIP로 오는 패킷이 새 Master로 전달된다.

이 전환은 보통 수 초 안에 완료된다.

## HAProxy와의 관계

HAProxy는 VRRP를 직접 구현하지 않는다. Keepalived가 VRRP를 담당하고, HAProxy는 트래픽 처리만 한다. 둘이 같은 서버에서 함께 실행된다.

참고: keepalived.md, vip.md, active-passive.md
