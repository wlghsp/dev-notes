# dnf 오프라인 로컬 레포 구성 — MariaDB 설치 사례

인터넷이 안 되는(폐쇄망/오프라인) 서버에 dnf로 패키지를 설치하려면, 로컬 디스크의 디렉토리를 저장소(repo)처럼 취급하도록 만들어야 한다.

---

## 핵심 원리

dnf는 원래 원격 저장소(baseurl이 http://...)에서 rpm을 받아온다. 오프라인 환경에서는 이게 불가능하므로, **로컬 디렉토리를 저장소로 등록**해서 dnf가 그 디렉토리 안의 rpm 파일들과 메타데이터(repodata)를 읽어 설치하게 만든다.

---

## 준비 절차

### 1. 인터넷 되는 환경에서 rpm 다운로드

동일한 OS 버전(RHEL/CentOS/Rocky 등, 버전까지 맞춰야 함 — 예: 8.x끼리)의 머신에서 필요한 패키지와 의존성을 전부 받는다.

```bash
# mariadb-server와 모든 의존 패키지를 재귀적으로 다운로드
dnf download --resolve --alldeps mariadb-server -y --destdir=/tmp/mariadb-rpms
```

- `--resolve --alldeps`: 의존성 패키지까지 전부 받음
- `dnf download` 플러그인(`dnf-plugins-core` 또는 `dnf-utils`)이 필요할 수 있음

의존성 누락 걱정되면 실제 설치를 `--downloadonly`로 테스트하고 그 캐시를 긁어가는 방법도 흔히 씀:

```bash
dnf install --downloadonly --downloaddir=/tmp/mariadb-rpms mariadb-server
```

### 2. rpm 파일들을 오프라인 서버로 이동

USB, scp, 사내망 등으로 `/tmp/mariadb-rpms` 디렉토리 전체를 오프라인 서버로 복사.

### 3. 로컬 저장소 메타데이터 생성 (createrepo)

rpm 파일 더미만으로는 dnf가 저장소로 인식하지 못한다. **repodata**(패키지 목록, 의존성 정보를 담은 XML/sqlite 인덱스)가 있어야 한다.

```bash
# createrepo_c 패키지 필요 (이것도 오프라인이면 미리 받아둬야 함)
createrepo_c /path/to/local-repo-dir
```

실행하면 해당 디렉토리 안에 `repodata/` 폴더가 생긴다. 이게 있어야 저장소로 인식된다.

### 4. .repo 파일 등록

`/etc/yum.repos.d/local.repo` 생성:

```ini
[local-mariadb]
name=Local MariaDB Repo
baseurl=file:///path/to/local-repo-dir
enabled=1
gpgcheck=0
```

- `baseurl=file://...`: http가 아니라 로컬 파일 경로 — 오프라인 설치의 핵심
- `gpgcheck=0`: 사내 서명이 없으면 보통 꺼둠 (또는 GPG 키를 별도로 import)

### 5. 설치

```bash
dnf clean all
dnf makecache
dnf install mariadb-server
```

dnf가 `local-mariadb` 레포의 repodata를 읽어 의존성 트리를 계산하고, 같은 디렉토리의 rpm 파일들을 설치한다.

---

## 전체 흐름

```
[인터넷 O 서버] dnf download (의존성 포함)
        ↓ (USB/scp로 이동)
[오프라인 서버] rpm 파일들을 특정 디렉토리에 배치
        ↓
createrepo_c 로 repodata 생성
        ↓
.repo 파일에 baseurl=file:// 로 등록
        ↓
dnf install mariadb-server
```

---

## 참고
- 관련 용어: createrepo, repodata, baseurl, gpgcheck
