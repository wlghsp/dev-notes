# two-phase-commit

**"두 개의 로그를 원자적으로 기록하기 위한 내부 프로토콜"**

MySQL InnoDB에서 커밋 시 redo log와 binlog가 모두 기록되어야 한다. 둘 중 하나만 기록된 채로 크래시가 나면 두 로그가 불일치하게 되고, 복제 환경에서 Master와 Slave 데이터가 달라진다.

이를 막기 위해 InnoDB는 내부적으로 2PC를 사용한다.

---

## 커밋 시 동작 순서

```
1. Prepare 단계
   → redo log에 "prepare" 상태로 기록
   → 아직 커밋 확정 아님

2. binlog 기록
   → binlog에 트랜잭션 이벤트 기록 + fsync

3. Commit 단계
   → redo log에 "commit" 상태로 업데이트 + fsync
   → 클라이언트에 완료 응답
```

---

## 크래시 시 복구 판단 기준

크래시 복구 시 redo log의 상태를 보고 판단한다.

- redo log가 commit 상태 → 정상 커밋으로 인정, 복구 적용
- redo log가 prepare 상태 + binlog에 해당 트랜잭션 있음 → 커밋으로 인정
- redo log가 prepare 상태 + binlog에 해당 트랜잭션 없음 → 롤백 처리

binlog 기록 완료 여부가 커밋 인정의 기준이 된다. binlog까지 기록됐으면 이미 Slave에 전파됐을 수 있기 때문에 커밋으로 처리해야 일관성이 유지된다.

---

## 한 줄 요약

> redo log와 binlog를 원자적으로 기록하기 위한 내부 프로토콜. binlog 기록 완료 여부가 커밋 인정의 기준.

참고: redo-log.md, binlog.md
