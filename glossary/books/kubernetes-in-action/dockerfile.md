# Dockerfile
참고: container-image.md, container-registry.md

---

컨테이너 이미지를 빌드하기 위한 명세 파일이다. Docker가 이 파일을 읽어 이미지를 만든다. Dockerfile의 각 지시어(directive)마다 이미지 레이어가 하나씩 생성된다.

## 주요 지시어

- `FROM` — 베이스 이미지 지정. 이 이미지의 레이어 위에 새 레이어를 쌓는다
- `COPY` — 빌드 컨텍스트의 파일을 이미지 안으로 복사
- `RUN` — 빌드 중에 명령어 실행. 실행 결과가 새 레이어로 저장된다
- `ENTRYPOINT` — 컨테이너 시작 시 실행할 명령 지정

## 레이어와 이미지 크기

`RUN`으로 파일을 생성한 뒤 이후 지시어에서 삭제해도 이미지 크기는 줄지 않는다. 삭제는 새 레이어에만 기록되고, 원본 레이어는 그대로 남기 때문이다. 임시 파일은 같은 `RUN` 명령 안에서 생성과 삭제를 함께 처리해야 한다.

## 빌드 과정

`docker build` 명령을 실행하면 Docker CLI가 현재 디렉토리 전체를 Docker 데몬에 전송한다. 빌드는 CLI가 아닌 데몬이 수행한다. macOS/Windows 환경에서는 데몬이 Linux VM 안에서 동작한다.

> 📷 Figure 2.14 (책 p.62) — Dockerfile로 이미지를 빌드하는 과정
