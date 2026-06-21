# container-registry (컨테이너 레지스트리)
참고: container-image.md, dockerfile.md

---

컨테이너 이미지를 저장하고 배포하는 저장소다. 빌드한 이미지를 레지스트리에 push하면, 다른 컴퓨터에서 pull해서 실행할 수 있다. 이미지 레이어 단위로 전송되므로 이미 가진 레이어는 다운로드하지 않는다.

대표적인 공개 레지스트리로 Docker Hub(hub.docker.com)가 있고, Quay.io, Google Container Registry 등도 사용된다. 사내 프라이빗 레지스트리를 직접 운영하는 경우도 많다.

## 이미지 태그

같은 이미지 이름 아래 여러 버전/변형을 태그로 구분한다. 태그를 명시하지 않으면 `latest`가 기본값이다. `redis:5.0.7-alpine`처럼 버전과 베이스 이미지 변형을 태그에 담는 것이 일반적이다.

`docker run` 명령은 로컬에 이미지가 없을 때만 레지스트리에서 pull한다. 한 번 받아두면 이후에는 로컬 캐시를 사용한다.
