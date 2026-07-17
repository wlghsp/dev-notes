# 학습 순서 — OCP/OpenStack/스토리지/네트워킹 용어

레드햇 가상화·컨테이너·스토리지·네트워킹 생태계 용어를 커뮤니케이션에서 헷갈리지 않을 정도로 정리한 문서들이다. 아래 순서로 읽으면 각 파일이 앞 파일의 개념을 전제로 이어진다.

## 1단계 — 축 잡기

1. rhv-ocp-osp-overview.md — 먼저 읽는다. "회사(레드햇 vs VMware)"와 "인프라 종류(VM vs 컨테이너)"라는 두 축을 여기서 세운다. 이 축이 나머지 모든 파일을 이해하는 기준이 된다

## 2단계 — 개별 용어 (개요에서 언급된 순서대로)

2. rhv.md — 단종된 레거시 VM 가상화 플랫폼
3. rhov.md — RHV의 후속. "OCP 위에서 VM을 직접 돌린다"는 패턴이 처음 등장하는 지점
4. ocp.md — 컨테이너 오케스트레이션 플랫폼. rhov.md가 전제로 삼는 기반
5. kubernetes-vs-ocp.md — ocp.md가 "쿠버네티스 배포판"이라고만 짧게 짚은 것을, 순정 쿠버네티스와 정확히 뭐가 다른지(설치 도구, 인증, 이미지 빌드 등)로 풀어준다. 오브젝트/API 자체는 그대로라는 게 핵심
6. openstack.md — RHOSP가 "OpenStack 배포판"이라고만 짧게 짚을 것의 실체. OpenStack 자체가 여러 컴포넌트(Nova, Neutron, Cinder)로 이루어진 오픈소스 IaaS 프로젝트라는 것과, 왜 여러 회사가 각자 배포판을 내는지를 커널/배포판 비유로 풀어준다
7. provisioning.md — openstack.md에서 반복해서 나온 "프로비저닝"이라는 단어 자체를 짚는다. 단순히 "생성"이 아니라 자원 할당부터 설정, 사용 가능한 상태로 넘겨주기까지 전체 흐름을 가리킨다는 것
8. rhosp-osp.md — OpenStack 기반 VM/IaaS 플랫폼
9. rhoso.md — RHOSP를 OCP 위에서 서비스로 돌리는 최신 아키텍처. rhov.md와 같은 패턴(OCP가 기반)이 다시 나온다
10. vsphere-vcenter.md — 1단계에서 잡은 "VMware" 축의 실체. RHV/RHOSP와의 대응 관계를 확인한다. 대응은 한 층이 아니라 두 층(ESXi/vCenter↔KVM/RHV, vCloud Director↔OpenStack)이라는 점이 핵심
11. kvm.md — 여기서 RHV/RHOSP/RHOV가 이름과 관리 방식만 다를 뿐 결국 공유하는 최하위 하이퍼바이저 엔진이 KVM이라는 걸 정리한다. RHV, RHOSP, RHOV, ESXi(vSphere)까지 다 읽은 뒤라야 이 파일의 대응 관계가 온전히 이해된다
12. kubevirt.md — rhov.md에서 "KubeVirt 기반"이라고만 짚었던 것의 실체. QEMU/KVM 프로세스를 Pod(virt-launcher) 안에 넣어 VM을 쿠버네티스 오브젝트로 만드는 방식을 kvm.md·pod.md 개념 위에서 풀어준다. OCP와 KVM이 같은 노드 위에서 나란한 관계라는 점, RHOSO와의 구조적 차이(도구만 컨테이너화 vs VM 자체가 Pod), 그리고 이 방식을 택한 이유(성능이 아니라 운영 비용)까지 다룬다
13. container-vs-vm.md — 컨테이너와 VM의 격리 방식 자체가 근본적으로 다르다는 것(커널 공유 vs 커널 분리)을 정리한다. kubevirt.md에서 "Pod라는 틀은 같고 내용물만 다르다"고 한 게 정확히 무슨 뜻인지 여기서 확인된다

## 3단계 — 제3진영과의 비교

14. proxmox.md — 레드햇도 VMware도 아닌 제3의 가상화 플랫폼. OpenStack이 대규모 멀티테넌트 IaaS를 노리는 분산 시스템이라 컴포넌트/장애 지점이 많은 반면, Proxmox는 노드 몇 대 규모를 단순하게 관리하는 도구라는 "규모 차이" 축을 여기서 짚는다

## 4단계 — 스토리지

15. storage-overview.md — 스토리지 쪽 개요. "Ceph는 저장소 하나, ODF/Cinder는 플랫폼별 연결 계층"이라는 축. 2단계에서 익힌 OCP/OpenStack 구도가 그대로 재사용된다
16. ceph.md — 실제 데이터를 저장하는 백엔드
17. odf.md — OCP(컨테이너)용 연결 계층
18. cinder.md — OpenStack(VM)용 연결 계층

## 5단계 — 네트워킹

19. networking-overview.md — 네트워킹 쪽 개요. "OVN은 엔진 하나, Neutron/OVN-Kubernetes는 플랫폼별 연결 계층"이라는 축 — 4단계 스토리지와 동일한 패턴이 한 번 더 반복된다
20. neutron.md — OpenStack(VM)용 네트워크 관리 서비스
21. ovn-kubernetes.md — OCP(컨테이너)용 CNI 플러그인

## 읽고 나서 확인할 것

각 개요 문서(rhv-ocp-osp-overview.md, storage-overview.md, networking-overview.md)의 "한 줄 구분법" 문단만 다시 봐도 커뮤니케이션 중에 이름이 나왔을 때 바로 축을 떠올릴 수 있는지 스스로 점검해본다. 스토리지와 네트워킹 두 개요가 결국 같은 축(엔진 하나 + 플랫폼별 연결 계층)의 반복이라는 걸 알아채면, 그리고 proxmox.md에서 짚은 "규모 차이"까지 구분할 수 있으면 이 폴더는 다 이해한 것이다.
