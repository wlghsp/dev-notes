# RHOSP / OSP (Red Hat OpenStack Platform)

RHOSP는 레드햇이 배포하는 OpenStack 기반 플랫폼이다. 커뮤니케이션에서는 흔히 OSP로 줄여 부른다.

## OpenStack이 하는 일

RHOSP는 OpenStack이라는 오픈소스 IaaS 프로젝트를 레드햇이 엔터프라이즈용으로 패키징하고 지원하는 배포판이다. OpenStack 자체가 뭘 하는 프로젝트인지는 openstack.md 참고.

## 무엇을 관리하는가

VM 기반 인프라다. RHV와 마찬가지로 컨테이너가 아니라 가상머신을 다루지만, RHV가 단일 하이퍼바이저 관리 툴에 가깝다면 RHOSP는 그보다 큰 범위의 클라우드 인프라(네트워크, 스토리지, 멀티테넌시 포함)를 다루는 IaaS 계층이라는 차이가 있다.

## 최신 아키텍처: RHOSO로의 전환

전통적인 RHOSP는 OpenStack 컴포넌트들이 별도 VM이나 베어메탈 위에서 직접 돌았다. 최신 아키텍처인 RHOSO(Red Hat OpenStack Services on OpenShift)부터는 이 OpenStack 컴포넌트 자체가 컨테이너화되어 OCP 클러스터 위에서 돈다. 그래서 "OSP"라는 이름을 들었을 때 전통적 방식인지 OCP 기반 RHOSO인지 구분해서 들어야 한다.

## RHOV로 충분한데 왜 굳이 RHOSP/RHOSO를 얹나

RHOV(OCP 위의 VM 기능)만으로도 VM을 띄우는 것 자체는 된다. RHOSP/RHOSO가 필요한 이유는 VM 하나를 띄우는 게 아니라, VM들을 위한 완전한 IaaS 계층 — 테넌트별 가상 네트워크(Neutron)와 서브넷/보안 그룹, 테넌트별 스토리지 쿼터(Cinder), 셀프서비스 프로비저닝 API, 프로젝트 단위 과금/쿼터 — 이 필요하기 때문이다. RHOV에는 이런 멀티테넌트 자원 모델이 없다.

즉 RHOSO가 존재하는 이유는 "RHOV로 될 걸 굳이 OpenStack으로 하자"가 아니라, 이미 OpenStack이 필요한 조직(여러 팀/고객에게 독립된 가상 데이터센터를 제공해야 하는 사내 클라우드, 통신사 NFV 워크로드 등)이 그 컨트롤 플레인을 별도 베어메탈 대신 OCP 위에 올려서 쿠버네티스 운영 모델로 통합하려는 것이다. 참고: rhov.md, networking-overview.md

## 왜 더 단순한 대안(Proxmox 등)과 자주 비교되나

OpenStack은 대규모 멀티테넌트 IaaS를 목표로 설계된 분산 시스템이라, 컴포넌트 수만큼 장애 지점도 많고 장애 원인 추적이 어렵다. Proxmox VE처럼 노드 몇 대 규모를 단순하게 관리하는 도구와 비교하면 안정성 체감이 다르게 느껴지는데, 이는 결함 차이가 아니라 애초에 풀려는 문제의 규모 차이다. 자세한 내용은 proxmox.md 참고.

참고: rhv-ocp-osp-overview.md, rhoso.md, ocp.md, rhov.md, proxmox.md, openstack.md
