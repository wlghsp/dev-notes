# Cinder

Cinder는 OpenStack 안에서 VM에 블록 스토리지(가상 디스크)를 붙여주는 컴포넌트다. OpenStack의 컴퓨트 서비스인 Nova가 VM을 만들면, Cinder가 그 VM이 쓸 디스크 볼륨을 만들어서 연결한다.

## 무엇을 하는가

Cinder 자체가 데이터를 저장하는 것은 아니다. 실제 저장은 백엔드(주로 Ceph)가 담당하고, Cinder는 "이 VM에 100GB 볼륨을 만들어서 붙여줘" 같은 요청을 받아 백엔드에 전달하고 관리하는 API 계층이다.

## 커뮤니케이션에서 헷갈리는 지점

컨테이너 쪽의 ODF와 대응되는 역할이지만, Cinder는 VM(OpenStack) 전용이고 ODF는 Pod(OCP) 전용이다. "스토리지가 안 붙는다"는 문제가 보고되면 VM 환경인지 컨테이너 환경인지부터 확인해야 Cinder 쪽 이슈인지 ODF 쪽 이슈인지 갈린다.

참고: storage-overview.md, ceph.md, odf.md, rhosp-osp.md
