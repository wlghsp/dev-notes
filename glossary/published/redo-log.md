# redo-log

**"커밋된 변경을 디스크에 보장하기 위한 순차 기록 파일"**

데이터베이스가 변경 내용을 데이터 파일에 바로 쓰지 않고, 먼저 Redo Log에 순차적으로 기록한다. 서버가 갑자기 죽어도 Redo Log를 재생(replay)해서 커밋된 트랜잭션을 복구할 수 있다.

WAL(Write-Ahead Logging)이라고도 부른다. "데이터 파일보다 로그를 먼저 쓴다"는 원칙이다.

---

## 실제로 어디에 저장되나

디스크의 파일에 적힌다.

- MySQL 8.0.30 미만 — `ib_logfile0`, `ib_logfile1`
- MySQL 8.0.30 이상 — `#ib_redo0`, `#ib_redo1` 형태로 변경

순환 구조로 동작한다. 파일이 꽉 차면 체크포인트가 지나간 오래된 영역부터 덮어쓴다. undo log와 달리 purge 스레드가 없고, 오래된 내용은 자동으로 덮어써진다.

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

## 크래시 복구에서 redo log의 역할

redo log는 데이터 페이지뿐 아니라 undo page도 보호한다. undo log도 Buffer Pool을 거쳐 디스크에 내려가기 때문에, 크래시 시점에 undo page가 디스크에 반영 안 됐을 수 있다.

```
크래시 복구 순서:
1. redo log → 커밋된 변경 재적용 (undo page 포함)
2. undo log → 커밋 안 된 트랜잭션 롤백
```

undo log가 커밋 시점에 fsync를 강제하지 않아도 되는 이유가 여기 있다. redo log가 먼저 undo page를 복구해주기 때문에, 그 다음 undo log로 롤백을 수행할 수 있다.

---

## 한 줄 요약

> COMMIT = Redo Log fsync 완료. 데이터 파일 반영은 그 이후. undo page 복구도 redo log가 담당한다.

참고: dirty-page.md, undo-log.md
