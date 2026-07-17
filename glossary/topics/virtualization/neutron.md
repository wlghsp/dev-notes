# Neutron

Neutron은 OpenStack 안에서 네트워크를 관리하는 컴포넌트다. VM을 만들 때 그 VM이 쓸 가상 네트워크, 서브넷, 라우터, 보안 그룹(방화벽 규칙)을 만들고 연결하는 역할을 한다.

## 무엇을 하는가

Nova가 VM을 만들고 Cinder가 그 VM에 디스크를 붙이는 동안, Neutron은 그 VM이 통신할 네트워크 인터페이스를 만들어서 붙인다. 사용자가 "이 프로젝트 전용 사설 네트워크 10.0.1.0/24를 만들고, 외부로 나가는 라우터도 연결해줘" 같은 요청을 하면 Neutron이 그 네트워크 토폴로지를 실제로 구성한다. 참고: network-cidr.md

## 내부 구현: OVN 드라이버

Neutron은 여러 백엔드 드라이버를 선택할 수 있는데, 최신 RHOSP는 OVN을 드라이버로 쓴다. 그래서 "Neutron이 네트워크를 관리한다"는 말과 "OVN이 실제로 브리지와 흐름 규칙을 만든다"는 말이 같은 상황을 가리키는 경우가 많다 — Neutron은 API/관리 계층이고, OVN은 그 아래에서 실제로 동작하는 엔진이다.

참고: networking-overview.md, ovn-kubernetes.md, rhosp-osp.md
