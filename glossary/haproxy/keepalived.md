# Keepalived

리눅스에서 고가용성을 구현하기 위한 데몬. VRRP 프로토콜을 이용해 여러 서버 중 하나에 VIP(Virtual IP)를 할당하고, 장애 시 자동으로 다른 서버로 VIP를 이전(failover)한다.

HAProxy Active-Passive 이중화에서 Keepalived가 failover를 담당한다.

## 역할

HAProxy는 백엔드 서버의 장애를 감지하고 트래픽을 분산하는 역할을 한다. 그러나 HAProxy 자체가 죽었을 때는 HAProxy가 스스로 대응할 수 없다. Keepalived가 그 역할을 맡아 HAProxy 노드 간 failover를 처리한다.

## 동작 흐름

1. Active HAProxy + Keepalived가 VIP를 소유한다.
2. Keepalived가 주기적으로 Passive 노드에 VRRP heartbeat를 보낸다.
3. Passive 노드가 heartbeat를 받지 못하면 Active가 죽었다고 판단한다.
4. Passive가 VIP를 자신에게 가져온다 (ARP 브로드캐스트로 알림).
5. 이후 클라이언트 요청은 새 Active(이전 Passive)로 간다.

## 설정 예시 (Active 노드)

```
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100            # Active가 더 높은 priority
    advert_int 1            # 1초마다 heartbeat

    virtual_ipaddress {
        192.168.1.100       # VIP
    }
}
```

Passive 노드는 동일한 설정에서 `state BACKUP`, `priority 90`으로 설정한다.

## HAProxy 상태 감시

Keepalived는 단순히 노드 생존 여부만 보는 게 아니라, 스크립트를 실행해 HAProxy 프로세스가 살아있는지 확인할 수도 있다. HAProxy 프로세스가 죽으면 VIP를 넘긴다.

```
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight -20
}
```

참고: active-passive.md, vrrp.md, vip.md
