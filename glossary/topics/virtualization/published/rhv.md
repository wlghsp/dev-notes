# RHV (Red Hat Virtualization)

RHV는 레드햇의 서버 가상화 플랫폼이다. KVM 하이퍼바이저 위에 VM을 올려서 관리하는 제품으로, VMware vSphere/vCenter와 직접 경쟁하던 위치에 있었다.

## 무엇을 관리하는가

VM이다. 컨테이너가 아니라 전통적인 하이퍼바이저 기반 가상머신을 프로비저닝하고 운영한다. RHEV(Red Hat Enterprise Virtualization)라는 옛 이름으로 불리기도 했다.

## 현재 상태

단종됐다. 레드햇은 RHV의 후속으로 RHOV(Red Hat OpenShift Virtualization)를 밀고 있는데, 이는 별도의 하이퍼바이저 플랫폼이 아니라 OCP(쿠버네티스) 클러스터 위에서 KubeVirt로 VM을 돌리는 방식이다. 그래서 커뮤니케이션에서 "RHV 대체제가 뭐냐"고 물으면 답은 "새로운 VM 전용 플랫폼"이 아니라 "OCP 위의 VM 기능(RHOV)"이라는 점이 헷갈리는 부분이다.

참고: rhv-ocp-osp-overview.md, ocp.md, rhov.md
