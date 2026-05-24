# undo-log

**"트랜잭션이 수정하기 전 데이터를 보관하는 로그"**

InnoDB가 row를 수정할 때 이전 버전을 undo log에 저장한다. ROLLBACK 시 원상복구에 쓰이고, MVCC에서 과거 버전 데이터를 읽을 때도 이걸 따라간다.

---

## 언제 기록되나

DML이 실행되는 순간, 실제 데이터 페이지를 바꾸기 전에 기록된다.

- UPDATE → 변경 전 row 값을 undo log에 기록
- DELETE → 삭제 전 row를 undo log에 기록 (실제 삭제는 purge가 나중에 처리)
- INSERT → 삽입된 row의 PK만 기록 (롤백 시 해당 row를 지우면 되므로)

커밋 여부와 관계없이 DML 실행 시점에 즉시 쓰인다.

---

## 두 가지 역할

**ROLLBACK 복구**

트랜잭션이 ROLLBACK되면 undo log에 저장된 이전 값으로 되돌린다.

```
UPDATE users SET name='Kim' WHERE id=1
  → undo log에 name='Lee' 저장 (이전 값)

ROLLBACK
  → undo log에서 name='Lee' 꺼내서 복원
```

**MVCC 이전 버전 제공**

다른 트랜잭션이 수정 중인 row를 읽어야 할 때, undo log 체인을 따라가서 자신의 Read View 기준에 맞는 버전을 찾는다.

```
트랜잭션 B의 Read View: trx_id 80 이하만 볼 수 있음

현재 row: name='Kim', trx_id=100  ← 안 보임
undo log: name='Lee', trx_id=75   ← 보임, 이걸 반환
```

---

## undo log 체인

row가 여러 번 수정되면 undo log가 체인 형태로 쌓인다. 각 버전은 이전 버전을 가리키는 포인터(`roll_ptr`)를 가지고 있어서 체인이라고 부른다.

```
현재: name='Kim',   trx_id=100
  ↓ roll_pointer
이전: name='Lee',   trx_id=75
  ↓ roll_pointer
이전: name='Park',  trx_id=30
```

MVCC는 Read View 기준을 만족하는 버전이 나올 때까지 이 체인을 따라 내려간다.

---

## 실제로 어디에 저장되나

바로 디스크에 쓰지 않는다. DML이 실행되면 undo log를 Buffer Pool의 undo page에 먼저 쓰고, 이후 일반 데이터 페이지처럼 백그라운드에서 디스크로 flush된다.

최종적으로 디스크 파일에 저장된다.

- MySQL 5.x — `ibdata1` 파일 안에 데이터와 함께 저장
- MySQL 8.0 — `undo_001`, `undo_002` 별도 파일로 분리

row가 수정될 때마다 이전 버전이 이 파일에 순서대로 쌓인다. purge 스레드가 공간을 해제할 때 파일 자체를 지우는 게 아니라, 파일 내부에서 해당 영역을 "재사용 가능" 상태로 표시한다.

undo log 자체는 커밋 시점에 반드시 디스크에 sync할 필요가 없다. 크래시가 나면 redo log가 undo page를 먼저 복구하고, 그 undo log를 써서 미완료 트랜잭션을 롤백하는 순서로 처리하기 때문이다.

```
크래시 복구 순서:
1. redo log → 커밋된 변경 재적용 (undo page 포함)
2. undo log → 커밋 안 된 트랜잭션 롤백
```

---

## 언제 삭제되나

COMMIT한다고 바로 지워지지 않는다. 해당 버전을 참조하는 트랜잭션이 모두 끝나야 purge 스레드가 삭제한다.

오래된 트랜잭션이 계속 열려있으면 undo log가 계속 쌓인다. 이게 MySQL에서 "long running transaction" 이 위험한 이유 중 하나다.

---

## Redo Log와의 차이

- Redo Log — 커밋된 변경을 디스크에 보장하기 위한 로그. 복구용
- Undo Log — 수정 전 데이터를 보관하는 로그. ROLLBACK과 MVCC용

방향이 반대다. Redo는 "다시 적용", Undo는 "되돌리기".

---

## 한 줄 요약

> undo log = 수정 전 데이터 보관. ROLLBACK 복구와 MVCC 과거 버전 읽기에 사용.

참고: mvcc.md, redo-log.md
