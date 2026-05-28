# HAProxy 학습 순서

기존 블로그(network-haproxy.md)에서 frontend/backend 구조, ACL, 헬스체크, 세션 고정, TCP vs HTTP 모드를 다뤘다. 이 순서는 그 위에 쌓이는 심화 개념 기준이다.

---

## 1단계 — 로드밸런싱 알고리즘

어떤 기준으로 서버를 선택하는가.

1. roundrobin.md — 순서대로 분배, weight 조정
2. leastconn.md — 현재 연결 수 기준, 오래 유지되는 연결에 적합
3. source.md — IP 해싱으로 서버 고정, TCP 모드에서 세션 고정
4. uri.md — URI 해싱, 캐시 서버 앞단에 유용

---

## 2단계 — 연결 관리

연결이 어떻게 제한되고 대기하는가.

5. timeout.md — connect / client / server / queue 타임아웃 구간별 의미
6. maxconn.md — global / frontend / server 레벨 최대 연결 수
7. queue.md — 서버 꽉 찼을 때 대기열 동작

---

## 3단계 — 세션과 TLS

클라이언트 고정과 암호화 처리.

8. sticky-session.md — 쿠키 기반 세션 고정, source와의 차이
9. ssl-termination.md — HAProxy에서 TLS 종료, 백엔드는 평문
10. ssl-passthrough.md — TLS 그대로 통과, end-to-end 암호화
11. sni.md — 하나의 IP에서 도메인별 인증서 선택

---

## 4단계 — 고가용성

HAProxy 자체가 죽지 않으려면.

12. vip.md — 클라이언트 접속점인 가상 IP
13. active-passive.md — HAProxy 이중화 구성
14. keepalived.md — failover를 담당하는 데몬
15. vrrp.md — Keepalived가 사용하는 VIP 관리 프로토콜

---

## 5단계 — 운영

트래픽 제어와 관찰.

16. rate-limiting.md — Stick Table로 IP별 요청 수 제한
17. connection-limiting.md — 동시 연결 수 제한, rate limiting과의 차이
18. retry-redispatch.md — 서버 실패 시 재시도 및 다른 서버로 전환
19. logging.md — 로그 포맷, 타이밍 필드로 성능 진단
