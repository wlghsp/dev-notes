# setup_cmd / grade_cmd

## 한 줄 요약

`setup_cmd`는 퀘스트 시작 전 환경을 준비하는 명령어, `grade_cmd`는 퀘스트 완료 여부를 판단하는 명령어.

---

## setup_cmd

### 왜 필요한가

퀘스트마다 독립된 컨테이너가 뜬다. 이전 퀘스트에서 만든 파일이 없다.
"파일 복사하기" 퀘스트를 풀려면 복사할 원본 파일이 미리 있어야 한다.
setup_cmd가 컨테이너 시작 직후, 사용자에게 터미널이 열리기 전에 그 환경을 만들어둔다.

### 실행 시점

```
컨테이너 시작 → setup_cmd 실행 완료 → connected 전송 → 터미널 열림
```

사용자는 setup_cmd가 끝난 뒤의 환경을 받는다.

### 형식

DB의 JSON 배열. `exec`으로 실행하므로 쉘 문법이 필요하면 `sh -c`를 앞에 붙인다.

```json
["touch", "/tmp/renamed.txt"]
["sh", "-c", "/usr/sbin/sshd && mkdir -p /tmp/sync_src"]
```

### setup_cmd가 없는 퀘스트

`NULL`이면 `runSetupCmd`가 아무것도 하지 않고 바로 넘어간다.

---

## grade_cmd

### 어떻게 동작하는가

"채점하기" 버튼을 누르면 백엔드가 실행 중인 컨테이너 안에서 grade_cmd를 실행한다.
**종료 코드 0이면 성공, 그 외면 실패.**

```
채점 버튼 클릭
    ↓
POST /grade { containerId, questId }
    ↓
gradeQuest() — DB에서 grade_cmd 조회
    ↓
container.exec(grade_cmd) → 실행
    ↓
ExitCode === 0 → passed: true / false 반환
```

### 형식

setup_cmd와 동일하게 JSON 배열.

```json
["test", "-f", "/tmp/backup.txt"]          // 파일 존재 여부
["grep", "-q", "done", "/tmp/answer.txt"]  // 파일 내용 확인
["sh", "-c", "test ! -f /tmp/renamed.txt"] // 파일 없는지 확인
```

### grade_cmd 작성 원칙

- 성공 조건만 확인한다. 실패 시 메시지 출력 불필요 (`-q` 플래그로 조용히).
- 파일 존재 → `test -f`, 디렉토리 → `test -d`, 내용 확인 → `grep -q`
- 쉘 조합이 필요하면 `["sh", "-c", "..."]` 형태로.

---

## 두 개의 차이 요약

| | setup_cmd | grade_cmd |
|---|---|---|
| 실행 시점 | 터미널 열리기 전 | 채점 버튼 클릭 시 |
| 목적 | 퀘스트 초기 환경 준비 | 퀘스트 완료 여부 판단 |
| 결과 | 무시 (종료만 기다림) | ExitCode 0/1 로 성공/실패 판단 |
| NULL 허용 | O | X (필수) |
