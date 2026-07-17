# ODF (OpenShift Data Foundation)

ODF는 OCP 클러스터 안에서 Ceph를 배포하고 운영해주는 오퍼레이터다. 예전 이름은 OCS(OpenShift Container Storage)였다.

## 무엇을 하는가

OCP 위에서 Pod가 쓸 영구 스토리지(PersistentVolume)가 필요할 때, 그 뒤에서 실제로 데이터를 저장하는 것이 ODF가 배포한 Ceph 클러스터다. 관리자는 Ceph를 직접 설치하고 튜닝하는 대신, ODF라는 쿠버네티스 오퍼레이터를 통해 선언적으로 스토리지를 프로비저닝한다.

## 커뮤니케이션에서 헷갈리는 지점

"ODF"라고 하면 별개의 스토리지 기술처럼 들리지만, 실체는 Ceph를 OCP 위에 편하게 얹는 배포/관리 계층이다. RHOV가 KVM을 OCP 위에 얹은 것과 같은 패턴이다 — 밑바탕 기술(Ceph, KVM)은 그대로고, 그 위에 OCP 친화적인 운영 계층이 씌워진 것이다.

참고: storage-overview.md, ceph.md, ocp.md, rhov.md
