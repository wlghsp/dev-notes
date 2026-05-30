# VIP (Virtual IP, 가상 IP)

특정 서버에 고정되지 않고 여러 서버 사이에서 이동 가능한 IP 주소. HAProxy 이중화에서 클라이언트가 실제로 접속하는 주소다.

## 왜 필요한가

HAProxy가 두 대 있을 때 클라이언트가 Active HAProxy의 실제 IP로 직접 접속하면, Active가 죽고 Passive로 전환될 때 클라이언트 설정을 바꿔야 한다.

VIP를 쓰면 클라이언트는 항상 같은 IP로 접속한다. Active가 바뀌어도 VIP만 새 서버로 이전되고 클라이언트는 아무것도 바꿀 필요 없다.

## 동작

VIP는 서버의 네트워크 인터페이스에 추가 IP로 할당된다. Active HAProxy의 eth0에 `192.168.1.100`(VIP)이 붙어있다. Active가 죽으면 Keepalived가 이 IP를 Passive의 eth0에 붙인다.

같은 네트워크의 장비들은 IP-MAC 매핑을 ARP 캐시로 관리한다. VIP가 이전되면 새 서버가 Gratuitous ARP를 날려 "VIP는 이제 내 MAC 주소"라고 알린다. 주변 장비들이 ARP 캐시를 업데이트하면 전환 완료.

## 클라이언트 관점

```
클라이언트 → VIP(192.168.1.100) → Active HAProxy → 백엔드 서버
                                        ↓ (장애)
클라이언트 → VIP(192.168.1.100) → Passive가 새 Active → 백엔드 서버
```

클라이언트 입장에선 항상 같은 IP. 전환 과정에서 진행 중인 연결은 끊길 수 있지만 새 연결은 정상적으로 처리된다.

참고: active-passive.md, keepalived.md, vrrp.md
