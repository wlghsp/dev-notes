# redo-log

**"커밋된 변경을 디스크에 보장하기 위한 순차 기록 파일"**

데이터베이스가 변경 내용을 데이터 파일에 바로 쓰지 않고, 먼저 Redo Log에 순차적으로 기록한다. 서버가 갑자기 죽어도 Redo Log를 재생(replay)해서 커밋된 트랜잭션을 복구할 수 있다.

WAL(Write-Ahead Logging)이라고도 부른다. "데이터 파일보다 로그를 먼저 쓴다"는 원칙이다.

---

## COMMIT과의 관계

COMMIT이 완료됐다는 건 Redo Log가 디스크에 fsync된 시점이다. 데이터 파일이 바뀐 시점이 아니다.

```
트랜잭션 중 UPDATE 실행
  → Redo Log buffer(메모리)에 변경 내용 기록
  → Buffer Pool에 dirty page 생성

COMMIT
  → Redo Log buffer → 디스크 fsync
  → 클라이언트에 완료 응답

이후 background
  → dirty page → 데이터 파일(.ibd) flush
```

Redo Log에 기록이 끝났으면 서버가 죽어도 재시작 시 복구 가능하기 때문에, 데이터 파일 반영을 기다리지 않고 응답을 보낸다.

---

## fsync가 중요한 이유

OS는 디스크 쓰기를 즉시 하지 않고 페이지 캐시에 버퍼링한다. `fsync`를 호출해야 OS 캐시가 실제 디스크에 내려간다. fsync 없이는 OS가 죽으면 기록이 사라진다.

---

## innodb_flush_log_at_trx_commit

COMMIT 시 Redo Log를 어느 수준까지 보장할지 설정한다.

- `1` (기본) — COMMIT마다 fsync. 가장 안전. 성능 비용 있음
- `2` — OS 버퍼까지만 씀. OS가 죽으면 최대 1초치 유실 가능
- `0` — InnoDB 버퍼에만 씀. InnoDB 프로세스 죽으면 유실 가능

---

## 한 줄 요약

> COMMIT = Redo Log fsync 완료. 데이터 파일 반영은 그 이후.

참고: dirty-page.md
