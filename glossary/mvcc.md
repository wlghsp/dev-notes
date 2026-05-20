# mvcc

**"같은 데이터의 여러 버전을 유지해서 읽기와 쓰기가 서로 블로킹하지 않게 하는 기법"**

MVCC(Multi-Version Concurrency Control). 트랜잭션 A가 데이터를 수정하는 동안 트랜잭션 B가 같은 데이터를 읽어야 할 때, B를 기다리게 하지 않고 수정 전 버전을 보여준다.

---

## 왜 필요한가

락(lock) 기반으로만 동시성을 제어하면 읽기도 쓰기를 기다려야 한다. 읽기가 많은 환경에서 성능이 급격히 떨어진다. MVCC는 "읽기는 락 없이, 쓰기는 새 버전 생성으로" 해결한다.

---

## InnoDB가 row에 숨겨놓는 컬럼

`SELECT *`나 `DESCRIBE`로는 보이지 않는다. InnoDB가 모든 row에 자동으로 붙이는 내부 메타데이터다.

```
실제 row 내부 구조

id | name | DB_TRX_ID | DB_ROLL_PTR       | DB_ROW_ID
1  | Kim  | 100       | → undo log 포인터 | (PK 없을 때만 생성)
```

- `DB_TRX_ID` — 이 row를 마지막으로 수정한 트랜잭션 ID
- `DB_ROLL_PTR` — undo log의 이전 버전을 가리키는 포인터
- `DB_ROW_ID` — PK가 없는 테이블에서만 생성되는 내부 식별자

MVCC가 동작할 수 있는 근거가 이 두 컬럼이다. row를 읽었을 때 `DB_TRX_ID`로 "내가 봐도 되는 버전인가"를 판단하고, 안 되면 `DB_ROLL_PTR`을 따라 undo log로 이동한다.

---

## 동작 방식

```
트랜잭션 A (trx_id=100): UPDATE users SET name='Kim' WHERE id=1
                          → 아직 COMMIT 안 한 상태

트랜잭션 B (trx_id=80):  SELECT * FROM users WHERE id=1
```

B가 row를 읽으면 dirty page에서 `DB_TRX_ID=100`을 확인한다.
100은 B의 Read View 기준(80)보다 크고 아직 활성 중이므로 보면 안 된다.
`DB_ROLL_PTR`을 따라 undo log로 이동해서 이전 버전을 찾는다.

```
dirty page:  name='Kim',  DB_TRX_ID=100  ← 읽었지만 "안 보이는 척"
undo log:    name='Lee',  trx_id=75      ← 75 < 80, 커밋됨 → 이걸 반환
             name='Park', trx_id=50      ← 더 이전 버전
```

B는 dirty page를 안 읽는 게 아니라, 읽고 나서 자신의 버전이 아님을 판단하고 undo log를 따라가는 것이다.

---

## 읽기 시점 기준 — Read View

트랜잭션이 시작할 때 "지금 활성 중인 트랜잭션 목록"을 스냅샷으로 찍는다. 이게 Read View다. 이후 읽기는 항상 이 스냅샷 기준으로 보인다.

```
트랜잭션 B 시작 시점의 Read View
  → 활성 트랜잭션: [95, 98, 100]
  → 이 ID들이 수정한 데이터는 보이지 않음
  → 80 이하 trx_id의 커밋된 데이터만 읽음
```

---

## undo log와의 관계

이전 버전 데이터는 undo log에 저장된다. MVCC가 과거 버전을 보여줄 수 있는 게 undo log 덕분이다. 트랜잭션이 ROLLBACK할 때도 undo log로 원상복구한다.

undo log는 아무도 참조하지 않게 되면 purge 스레드가 삭제한다. 오래된 트랜잭션이 계속 열려있으면 undo log가 쌓여서 디스크를 차지한다.

---

## COMMIT과의 관계

COMMIT해도 데이터가 즉시 "모두에게" 보이지 않는다. 이미 시작된 트랜잭션은 자신의 Read View 기준으로 계속 이전 버전을 읽는다. 새로 시작하는 트랜잭션부터 커밋된 데이터가 보인다.

---

## 한 줄 요약

> MVCC = 데이터의 여러 버전을 undo log로 유지. 읽기는 자신의 Read View 기준 버전을 봄. 읽기-쓰기가 서로 안 기다림.

참고: redo-log.md, dirty-page.md
