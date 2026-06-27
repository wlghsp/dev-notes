# WAL과 DB 로그 시스템

**한 줄 정의**: 데이터를 디스크에 쓰기 전에 반드시 로그를 먼저 기록하는 원칙 — Write-Ahead Logging.

---

## WAL이란

WAL(Write-Ahead Log)은 원칙이다. 특정 파일이 아니라 "데이터 변경 전에 로그를 먼저 써야 한다"는 규칙 자체를 말한다.

왜 이 순서가 중요한가? 디스크 쓰기는 원자적이지 않다. Page를 쓰는 도중에 서버가 죽으면 그 Page는 반만 쓰여진 깨진 상태가 된다. 데이터를 먼저 바꾸고 나서 죽으면 어디까지 쓰다 죽었는지 알 방법이 없다. 로그를 먼저 써두면 재시작 시 로그를 재적용해서 복구할 수 있다.

```
WAL 원칙:
  로그 기록 → (확인) → 실제 데이터 Page 변경

위반 시:
  데이터 Page 변경 → (서버 죽음) → 복구 불가
```

MySQL에서 WAL을 구현하는 로그는 세 가지다. 각각 담당하는 레이어와 목적이 다르다.

---

## Redo Log (InnoDB)

- 위치: Storage Engine 내부 — Recovery Manager
- 목적: Crash Recovery. 서버가 죽었다 살아났을 때 커밋된 트랜잭션을 재적용
- 형식: 물리적. 어떤 Page의 어떤 offset에 어떤 바이트를 썼는지 기록
- 구조: 순환 파일 (ib_logfile0, ib_logfile1). Checkpoint 이후 구간은 덮어씀

```
트랜잭션 커밋 흐름:
  변경 발생
  → Redo Log에 먼저 기록 (WAL 원칙)
  → Buffer Pool의 Dirty Page 변경
  → (나중에) Dirty Page → Disk flush
```

Checkpoint는 "여기까지는 Disk에 반영됐다"는 표시다. Crash 후 재시작 시 Checkpoint 이후 Redo Log만 재적용하면 된다 — 복구 범위를 줄인다.

Redo Log는 InnoDB 내부 포맷이라 외부에서 파싱하기 어렵다. InnoDB 버전에 종속된다.

---

## Undo Log (InnoDB)

- 위치: Storage Engine 내부 — Transaction Manager
- 목적: 두 가지. 롤백 + MVCC
- 형식: 논리적. "이 row의 이전 값은 이것이었다"를 기록
- 저장: InnoDB 시스템 테이블스페이스 또는 별도 undo tablespace

**롤백**: 트랜잭션이 실패하거나 명시적으로 ROLLBACK하면 Undo Log를 역방향으로 재적용해서 변경을 되돌린다.

**MVCC**: InnoDB는 Row를 수정할 때 기존 값을 덮어쓰지 않는다. 새 버전을 쓰고 이전 버전을 Undo Log에 남긴다. 다른 트랜잭션이 "이 시점의 스냅샷"을 읽을 때 Undo Log를 역으로 따라가서 해당 시점의 버전을 재구성한다.

```
Row 수정 흐름 (MVCC):
  UPDATE 실행
  → 현재 값을 Undo Log에 기록
  → Row를 새 값으로 변경 + 이전 Undo Log를 가리키는 포인터 유지

다른 트랜잭션이 이전 스냅샷 읽을 때:
  현재 Row → Undo Log 체인 역추적 → 해당 시점 버전 반환
```

Undo Log는 트랜잭션이 길수록, 동시 트랜잭션이 많을수록 쌓인다. Long transaction이 Undo Log를 오래 붙잡으면 공간 문제가 생긴다.

---

## Binlog (MySQL Server)

- 위치: Storage Engine 바깥 — MySQL Server Layer
- 목적: 복제(Replication) + 외부 전파 (CDC)
- 형식: 세 가지 모드 선택 가능
- 보존: 명시적으로 rotate/purge하기 전까지 유지

Binlog는 InnoDB만의 로그가 아니다. MySQL에서 실행된 모든 변경(어떤 Storage Engine이든)이 기록된다.

**Binlog 형식 세 가지:**

`STATEMENT` — 실행된 SQL 그대로 기록. 용량이 작다. 단점: NOW(), UUID(), RAND() 같은 non-deterministic 함수가 있으면 Replica에서 재실행 시 결과가 달라진다.

`ROW` — 어떤 row가 어떻게 바뀌었는지 before/after 이미지로 기록. 용량이 크다. CDC에서는 이 모드가 필수다 — before/after가 있어야 변경 이벤트를 정확히 캡처할 수 있다.

`MIXED` — 기본은 STATEMENT, non-deterministic 구문은 자동으로 ROW로 전환. 절충안이지만 CDC에는 부적합하다.

**CDC에서 Binlog를 쓰는 이유:**

Redo Log는 InnoDB 내부 물리 포맷이라 외부 도구가 읽기 어렵다. Binlog는 MySQL이 공개한 포맷이라 Debezium, Canal 같은 도구가 파싱할 수 있다. ROW 형식이면 어떤 row가 INSERT/UPDATE/DELETE됐는지 before/after 값을 그대로 꺼낼 수 있다.

---

## 세 로그 비교

**Redo Log**
- 레이어: Storage Engine / Recovery Manager
- 목적: Crash 복구
- 형식: 물리적
- 보존: 순환 덮어씀
- 외부 접근: 어려움 (InnoDB 내부 포맷)

**Undo Log**
- 레이어: Storage Engine / Transaction Manager
- 목적: 롤백 + MVCC
- 형식: 논리적
- 보존: 트랜잭션 종료 후 정리
- 외부 접근: 어려움

**Binlog**
- 레이어: MySQL Server Layer
- 목적: 복제 + CDC
- 형식: 논리적 (SQL/ROW)
- 보존: 명시적 purge 전까지 유지
- 외부 접근: 가능 (공개 포맷)

---

## 트랜잭션 커밋 시 전체 흐름

MySQL이 트랜잭션을 커밋할 때 Redo Log와 Binlog 사이에 내부 2PC(Two-Phase Commit)가 일어난다. 이 순서가 틀리면 Replica와 Leader 사이에 불일치가 생긴다.

```
COMMIT
  1. InnoDB: Redo Log에 Prepare 기록
  2. MySQL Server: Binlog에 이벤트 기록
  3. InnoDB: Redo Log에 Commit 기록

Crash 복구 시:
  - Redo Log에 Commit 있음 → 정상 커밋
  - Redo Log에 Prepare만 있고 Binlog에 있음 → 커밋으로 처리
  - Redo Log에 Prepare만 있고 Binlog에 없음 → 롤백
```

2번과 3번 사이에서 죽더라도, Binlog 유무로 커밋 여부를 판단할 수 있다. 이 덕분에 Replica가 읽은 Binlog와 Leader의 복구 결과가 일치한다.

---

## 관련 문서

- [DBMS 아키텍처 — Recovery Manager](db-dbms-architecture.md#recovery-manager)
- Phase 4: MVCC (Undo Log 상세)
- Phase 5: Recovery (Redo Log + Checkpoint 상세)
