# Network 트레이닝 로드맵
## "네트워크를 AI 없이 디버깅할 수 있는 개발자 되기"

> 특정 장비나 도구를 외우는 게 목표가 아니다.
> **패킷이 어디서 어디로 흐르는지, 왜 안 되는지 설명할 수 있는 수준**이 목표다.
> 탑다운 접근: K8s/실무 맥락에서 시작해서 TCP/IP 기초로 내려간다.

---

## 진행 원칙

```
실무에서 막혔던 경험을 개념으로 연결한다
"이렇게 설정했더니 됐다"가 아니라 "왜 됐는지" 설명할 수 있어야 완료다
AI 없이 트러블슈팅 흐름을 혼자 잡을 수 있으면 Phase 완료
```

---

## Phase 구성 전략

```
Phase 0~2 : K8s 네트워킹 — 실무에서 직접 부딪혔던 영역
Phase 3~4 : L4/L7, 로드밸런서, HAProxy — 왜 그게 어려웠는지 이해
Phase 5   : TCP/IP 기초 — 여기까지 오면 개념이 경험에 연결됨
Phase 6   : 실전 트러블슈팅 — 막혔을 때 어디서부터 보는가
```

---

## Phase 현황

- Phase 0 : K8s 네트워킹 기초 — Pod, Service, DNS ⬜
- Phase 1 : Ingress & OpenShift Route — L7 라우팅이 어떻게 동작하는가 ⬜
- Phase 2 : K8s 네트워크 정책 & IP 대역 — 왜 안 붙는가 ⬜
- Phase 3 : L4 vs L7 — 어디서 무엇이 결정되는가 ⬜
- Phase 4 : HAProxy & 로드밸런서 내부 — 왜 HAProxy가 어려운가 ⬜
- Phase 5 : TCP/IP 기초 — IP, TCP, DNS 흐름 ⬜
- Phase 6 : 실전 트러블슈팅 — 도구와 절차 ⬜

---

## 각 Phase 상세

### Phase 0: K8s 네트워킹 기초 — Pod는 어떻게 통신하는가
**파일**: `network-k8s-basics.md` (미작성)
**완료 기준**: "Pod가 다른 Pod에 요청을 보낼 때 패킷이 어떤 경로를 지나는지 설명 가능"
**핵심 질문**:
- Pod IP는 누가 어떻게 부여하는가? (CNI)
- ClusterIP Service는 실제로 어떻게 동작하는가? (kube-proxy, iptables/IPVS)
- Service에 요청을 보내면 실제로 어느 Pod로 가는가?
- K8s 내부 DNS는 어떻게 이름을 IP로 바꾸는가? (CoreDNS)

---

### Phase 1: Ingress & OpenShift Route — 외부 요청은 어떻게 들어오는가
**파일**: `network-ingress-route.md` (미작성)
**완료 기준**: "외부 요청이 Ingress를 거쳐 Pod까지 도달하는 흐름을 그림으로 설명 가능"
**핵심 질문**:
- Ingress와 Service의 차이는? 왜 둘 다 필요한가?
- Ingress Controller는 무엇이고 어디서 실행되는가?
- OpenShift Route가 Ingress와 다른 점은?
- TLS termination은 어디서 일어나는가?

---

### Phase 2: K8s IP 대역 & 네트워크 정책 — 왜 안 붙는가
**파일**: `network-k8s-ip-policy.md` (미작성)
**완료 기준**: "IP 대역 충돌이 왜 문제인지, NetworkPolicy로 무엇을 막을 수 있는지 설명 가능"
**핵심 질문**:
- Pod CIDR, Service CIDR, Node IP — 세 가지가 왜 겹치면 안 되는가?
- 센터 간 통신에서 IP 대역이 왜 중요한가?
- NetworkPolicy가 없으면 기본적으로 어떻게 동작하는가?
- VM ↔ K8s 클러스터 간 통신이 안 될 때 어디서부터 보는가?

---

### Phase 3: L4 vs L7 — 어디서 무엇이 결정되는가
**파일**: `network-l4-l7.md` (미작성)
**완료 기준**: "L4 로드밸런서와 L7 로드밸런서가 각각 무엇을 보고 결정하는지, 언제 어느 쪽을 쓰는지 설명 가능"
**핵심 질문**:
- L4는 무엇을 보는가? L7은 무엇을 더 보는가?
- K8s Service(ClusterIP/NodePort/LoadBalancer)는 L4인가 L7인가?
- Ingress가 L7인 이유는?
- 같은 포트로 여러 서비스를 분기하려면 왜 L7이 필요한가?

---

### Phase 4: HAProxy — 왜 그게 어려웠는가
**파일**: `network-haproxy.md` (미작성)
**완료 기준**: "HAProxy 설정 파일을 보고 frontend/backend 흐름을 머릿속에서 그릴 수 있음"
**핵심 질문**:
- HAProxy의 frontend / backend / ACL 구조는 무엇인가?
- TCP 모드와 HTTP 모드의 차이는?
- Health check는 어떻게 동작하는가?
- 실제 장애 시 HAProxy 로그에서 무엇을 봐야 하는가?

---

### Phase 5: TCP/IP 기초 — 여기까지 오면 연결된다
**파일**: `network-tcp-ip-basics.md` (미작성)
**완료 기준**: "TCP 연결이 맺어지고 끊어지는 과정을 설명할 수 있고, DNS 조회 흐름을 그릴 수 있음"
**핵심 질문**:
- IP 패킷은 어떻게 목적지를 찾아가는가? (라우팅)
- TCP 3-way handshake — 왜 3번인가?
- TIME_WAIT 상태는 왜 존재하는가? K8s에서 왜 문제가 되는가?
- DNS 조회는 어떤 순서로 일어나는가?

---

### Phase 6: 실전 트러블슈팅 — 막혔을 때 어디서부터 보는가
**파일**: `network-troubleshooting.md` (미작성)
**완료 기준**: "통신이 안 될 때 5분 안에 어디 문제인지 범위를 좁힐 수 있음"
**도구**:
- `kubectl exec`로 Pod 안에서 직접 curl/nslookup/ping
- `tcpdump`로 패킷 캡처
- `netstat` / `ss`로 연결 상태 확인
- HAProxy 로그 읽기
- K8s Events / Describe로 문제 범위 좁히기

---

## 참고
- 실무 맥락: HAProxy, K8s(OpenShift), VM 간 통신, 센터 간 통신
- 알고리즘 코드는 Python으로, 개념 예시는 Java/Spring 맥락으로
