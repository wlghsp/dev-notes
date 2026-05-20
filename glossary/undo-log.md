# undo-log

**"트랜잭션이 수정하기 전 데이터를 보관하는 로그"**

InnoDB가 row를 수정할 때 이전 버전을 undo log에 저장한다. ROLLBACK 시 원상복구에 쓰이고, MVCC에서 과거 버전 데이터를 읽을 때도 이걸 따라간다.

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

row가 여러 번 수정되면 undo log가 체인 형태로 쌓인다.

```
현재: name='Kim',   trx_id=100
  ↓ roll_pointer
이전: name='Lee',   trx_id=75
  ↓ roll_pointer
이전: name='Park',  trx_id=30
```

MVCC는 Read View 기준을 만족하는 버전이 나올 때까지 이 체인을 따라 내려간다.

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
