# Active-Passive (액티브-패시브)

HAProxy 자체를 이중화하는 구성. HAProxy가 단일 장애점(SPOF)이 되지 않도록 두 대를 두고, 하나는 Active(실제 트래픽 처리), 다른 하나는 Passive(대기)로 운영한다.

## 왜 필요한가

HAProxy가 아무리 백엔드를 이중화해도 HAProxy 자체가 죽으면 서비스가 중단된다. HAProxy를 이중화해야 완전한 고가용성이 된다.

## 동작

Active HAProxy가 VIP(Virtual IP)를 가지고 있다. 클라이언트는 VIP로 접속한다.

Active가 죽으면 Keepalived가 감지하고 Passive가 VIP를 가져간다(failover). 클라이언트 입장에서는 같은 IP로 계속 접속하므로 투명하게 전환된다.

```
Active HAProxy (192.168.1.1)  ┐
                               ├── VIP: 192.168.1.100 (클라이언트 접속점)
Passive HAProxy (192.168.1.2) ┘
```

## Keepalived + VRRP

Failover 메커니즘은 Keepalived가 담당한다. Keepalived는 VRRP 프로토콜로 두 노드 사이에 heartbeat를 보내고 VIP를 관리한다.

참고: keepalived.md, vrrp.md, vip.md

## Active-Active와의 차이

Active-Passive는 Passive가 대기 상태라 평소엔 Passive 자원이 낭비된다. 대신 구성이 단순하다.

Active-Active는 두 HAProxy 모두 트래픽을 처리한다. DNS 라운드 로빈이나 상위 로드밸런서로 분산한다. 자원 활용률이 높지만 구성이 복잡하다.

참고: keepalived.md, vrrp.md, vip.md
