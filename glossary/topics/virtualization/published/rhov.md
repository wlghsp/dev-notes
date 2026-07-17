# RHOV (Red Hat OpenShift Virtualization)

RHOV는 OCP(쿠버네티스) 클러스터 위에서 VM을 돌리는 레드햇의 기능이다. KubeVirt 프로젝트를 기반으로 한다. RHOSP가 OpenStack을 레드햇이 패키징한 것과 같은 관계다. 자세한 내용은 kubevirt.md 참고.

## RHV와의 관계

이름이 비슷해서 가장 헷갈리는 쌍이다. RHV(Red Hat Virtualization)는 단종된 독립 하이퍼바이저 플랫폼이고, RHOV(Red Hat OpenShift Virtualization)는 그 후속 기능이다. RHV는 자체 하이퍼바이저 관리 스택을 가진 별도 제품이었지만, RHOV는 별도 제품이 아니라 OCP 클러스터 안에서 동작하는 Operator 기반 확장 기능이다.

## 무엇을 관리하는가

VM이다. 다만 RHV처럼 전용 하이퍼바이저 플랫폼 위가 아니라, 컨테이너를 관리하던 OCP 클러스터의 같은 노드 위에서 VM이 Pod처럼 스케줄링된다. VM을 컨테이너와 동일한 방식(쿠버네티스 오브젝트, `oc`/`kubectl`)으로 다룰 수 있다는 것이 핵심 차이다. 참고: pod.md

## 왜 헷갈리는가

이름의 순서가 실제 관계를 그대로 반영한다 — RHV, OCP, RHOV 순서로 보면 "RHV라는 VM 플랫폼이 있었는데(RHV), 그게 컨테이너 플랫폼(OCP) 위에서 도는 기능으로 흡수됐다(RHOV)"는 흐름이다. RHOV를 얘기할 때는 관리 대상은 VM이지만 실행 기반은 OCP라는 점에서 RHOSO와 같은 패턴이다 — 다만 RHOSO는 OpenStack(IaaS 계층)을 얹은 것이고 RHOV는 VM 자체를 쿠버네티스 오브젝트로 직접 다룬다는 차이가 있다.

## RHOSP/RHOSO를 놔두고 RHOV를 쓰는 이유

RHOV는 VM 실행기에 가깝다. "이 스펙으로 VM 하나 띄워줘"라는 요청은 잘 처리하지만, 그 VM들을 위한 테넌트별 가상 네트워크, 서브넷, 보안 그룹, 스토리지 쿼터 같은 멀티테넌트 IaaS 자원 모델은 원래 범위 밖이다. 반대로 RHOSP/RHOSO는 Nova(컴퓨트)뿐 아니라 Neutron(네트워크), Cinder(스토리지) 같은 컴포넌트를 묶어서 여러 팀/고객에게 각자의 가상 데이터센터를 셀프서비스로 제공하는 IaaS 계층 전체를 담당한다.

그래서 선택 기준은 "OCP 위냐 아니냐"가 아니라 "VM만 필요한가, VM을 위한 완전한 IaaS 계층이 필요한가"다. 기존 OCP 클러스터 안에서 컨테이너와 VM 몇 대를 나란히 돌리고 싶은 정도면 RHOV로 충분하고, 여러 테넌트에게 독립된 네트워크/스토리지/프로비저닝 API를 제공해야 하는 사내 클라우드나 통신사 NFV 워크로드처럼 원래 OpenStack이 풀던 문제라면 RHOSP/RHOSO가 필요하다. 참고: rhosp-osp.md, networking-overview.md

OCP 클러스터의 노드 자체가 어디서 도는지(베어메탈인지 다른 하이퍼바이저의 VM인지)에 따라 RHOV의 성능에도 영향이 있다. 자세한 내용은 kvm.md 참고.

참고: rhv.md, ocp.md, rhoso.md, rhosp-osp.md, rhv-ocp-osp-overview.md, kubevirt.md, kvm.md
