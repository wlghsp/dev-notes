# vSphere / vCenter (VMware)

vSphere는 VMware의 서버 가상화 플랫폼이다. ESXi라는 자체 하이퍼바이저 위에 VM을 올려서 관리하며, RHV·RHOSP가 경쟁하던 대상이 바로 이것이다.

## vSphere와 vCenter의 관계

vSphere는 하이퍼바이저(ESXi)와 관리 도구를 합쳐 부르는 제품군 전체의 이름이고, vCenter는 그 안에서 여러 ESXi 호스트를 중앙에서 관리하는 관리 서버다. 즉 ESXi가 실제로 VM을 실행하는 하이퍼바이저라면, vCenter는 여러 ESXi 호스트를 묶어서 클러스터로 운영하게 해주는 컨트롤 플레인이다.

## 레드햇 진영과의 대응 관계

대응은 한 층이 아니라 두 층으로 나눠서 봐야 정확하다.

- 하이퍼바이저/단일 관리 레벨 — ESXi ↔ KVM, vCenter ↔ RHV. vCenter는 여러 ESXi 호스트를 묶어 VM을 마이그레이션·스케줄링하는 정도가 핵심이라, 셀프서비스 멀티테넌트 IaaS까지는 원래 범위가 아니다. 이 범위는 RHV와 거의 정확히 겹친다
- 멀티테넌트 IaaS 레벨 — VMware vCloud Director ↔ OpenStack(RHOSP). 여러 팀/고객에게 독립된 가상 데이터센터를 셀프서비스로 제공하는 계층은 vSphere가 아니라 vCloud Director가 담당한다. vCloud Director가 vSphere 위에 멀티테넌시·셀프서비스 포털·프로젝트별 쿼터를 얹는 역할은 OpenStack이 KVM 위에 얹는 역할과 같은 위치다. 참고: openstack.md

즉 "OpenStack에 대응하는 VMware 제품"을 물으면 vSphere/vCenter가 아니라 vCloud Director가 정확한 답이다. 다만 vCloud Director는 실무에서 OpenStack만큼 자주 언급되는 이름은 아니다.

VMware 쪽에는 OCP에 해당하는 컨테이너 오케스트레이션 전용 경쟁 제품이 따로 없다. VMware도 Tanzu 등으로 쿠버네티스 영역에 진출했지만, "레드햇 vs VMware" 축은 원래 VM 가상화 영역에서 형성된 경쟁 구도이지 컨테이너 영역까지 대칭적으로 겹치는 것은 아니다.

참고: rhv.md, rhosp-osp.md, rhv-ocp-osp-overview.md, kvm.md
