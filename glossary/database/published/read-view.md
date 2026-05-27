# read-view

**"트랜잭션이 어떤 버전까지 볼 수 있는지 판단하는 스냅샷 메타데이터"**

MVCC에서 undo log 체인을 타면서 "이 버전 읽어도 되나?"를 판단할 때 Read View를 기준으로 쓴다.

---

## 담고 있는 것

- `m_ids` — Read View 생성 시점에 활성 중인 트랜잭션 ID 목록
- `up_limit_id` — m_ids 중 가장 작은 trx_id (이보다 작으면 무조건 보임)
- `low_limit_id` — Read View 생성 시점의 다음 trx_id (이보다 크거나 같으면 무조건 안 보임)

---

## 판단 로직

undo log 체인의 각 버전에 붙은 `trx_id`를 Read View와 비교한다.

```
trx_id < up_limit_id         → 무조건 보임 (내 시작 전에 커밋된 것)
trx_id >= low_limit_id       → 무조건 안 보임 (내 시작 후에 생긴 것)
up_limit_id <= trx_id < low_limit_id 사이 → m_ids에 있으면 안 보임, 없으면 보임
```

m_ids에 있다는 건 Read View 생성 시점에 아직 커밋 안 된 트랜잭션이라는 뜻이다.

---

## 격리 수준별 생성 시점

Read View를 언제 만드느냐가 격리 수준의 핵심 차이다.

**REPEATABLE READ** — 트랜잭션 시작 시 한 번만 생성. 이후 쿼리는 같은 Read View를 재사용한다.

**READ COMMITTED** — 쿼리를 실행할 때마다 새로 생성. 매번 최신 커밋 기준으로 판단한다.

---

## 한 줄 요약

> Read View = MVCC가 "이 버전 보여줄지 말지"를 판단하는 기준. 생성 시점이 격리 수준을 결정한다.

참고: undo-log.md, mvcc.md, purge.md
