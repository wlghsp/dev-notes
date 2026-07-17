# OCP (OpenShift Container Platform)

OCP는 레드햇이 만든 쿠버네티스 배포판이다. 오픈소스 쿠버네티스에 인증, 이미지 빌드, 모니터링, 웹 콘솔 같은 엔터프라이즈 기능을 얹어서 패키징한 제품이다.

## 무엇을 관리하는가

컨테이너를 관리한다. VM이 아니라 컨테이너 오케스트레이션이 핵심이다. Pod, Deployment 같은 쿠버네티스 오브젝트를 그대로 쓰되, `oc` CLI와 웹 콘솔 같은 레드햇 고유의 운영 도구가 추가된다. 참고: pod.md

## 커뮤니케이션에서 헷갈리는 지점

"OCP 위에서 VM도 돌린다"는 말을 들으면 혼란스러울 수 있는데, 이건 OpenShift Virtualization이라는 별도 기능 때문이다. KubeVirt 기반으로 컨테이너 플랫폼 위에 VM을 얹어 돌리는 것이라, OCP 자체의 정체성(컨테이너 오케스트레이션)이 바뀐 것은 아니다. RHOSO도 같은 원리로 OCP 위에서 OpenStack 서비스가 도는 구조다.

순정 쿠버네티스와 정확히 뭐가 다른지는 kubernetes-vs-ocp.md 참고.

참고: rhv-ocp-osp-overview.md, rhoso.md, kubernetes-vs-ocp.md
