# isolation-level

**"트랜잭션이 다른 트랜잭션의 변경을 얼마나 볼 수 있는지 결정하는 수준"**

격리 수준이 높을수록 일관성은 높아지고 동시성은 낮아진다.

---

## 4단계

**READ UNCOMMITTED**

커밋 안 된 변경도 읽는다. undo log를 타지 않고 현재 버전을 그냥 읽는다.

- dirty read 발생: 다른 트랜잭션이 롤백하면 없던 데이터를 읽은 셈이 됨
- 실무에서 거의 안 씀

**READ COMMITTED**

커밋된 변경만 읽는다. 쿼리를 실행할 때마다 Read View를 새로 만든다.

- non-repeatable read 발생: 같은 쿼리를 두 번 실행하면 결과가 다를 수 있음
- Oracle 기본값, 많은 서비스에서 사용

**REPEATABLE READ**

트랜잭션 시작 시점에 Read View를 한 번만 만든다. 이후 쿼리는 같은 Read View를 재사용한다.

- 같은 SELECT를 몇 번 실행해도 결과가 같음
- phantom read 이론상 발생 가능하지만 InnoDB는 gap lock으로 방어
- MySQL InnoDB 기본값

**SERIALIZABLE**

MVCC 대신 잠금을 쓴다. 모든 SELECT에 shared lock이 걸린다.

- 완전한 직렬 실행과 동일한 결과 보장
- 동시성이 크게 떨어져 실무에서 드물게 사용

---

## READ COMMITTED vs REPEATABLE READ — 핵심 차이

둘 다 MVCC를 쓴다. undo log를 타는 것도 같다. 차이는 Read View를 언제 만드느냐다.

```
트랜잭션 B 시작
  → SELECT name  ← 첫 번째 쿼리, 결과: 'Lee'

트랜잭션 A: UPDATE name='Kim', COMMIT

  → SELECT name  ← 두 번째 쿼리
```

READ COMMITTED 결과:
```
첫 번째: 'Lee'   ← Read View 1 생성, A 아직 커밋 전
두 번째: 'Kim'   ← Read View 2 새로 생성, A가 이미 커밋됨
```

REPEATABLE READ 결과:
```
첫 번째: 'Lee'   ← 트랜잭션 시작 시 Read View 생성
두 번째: 'Lee'   ← 같은 Read View 재사용, A 커밋은 보이지 않음
```

Read View 생성 시점에 A가 커밋됐으면 보이고, 안 됐으면 안 보인다. 그 기준 시점을 고정하면 REPEATABLE READ, 매 쿼리마다 갱신하면 READ COMMITTED.

---

## 각 수준이 허용하는 이상 현상

**dirty read** — 커밋 안 된 데이터를 읽음
- READ UNCOMMITTED만 발생

**non-repeatable read** — 같은 쿼리를 두 번 실행했을 때 결과가 다름
- READ COMMITTED까지 발생, REPEATABLE READ부터 방지

**phantom read** — 같은 범위 쿼리를 두 번 실행했을 때 row 수가 다름
- REPEATABLE READ 이하에서 이론상 발생, SERIALIZABLE에서 방지
- InnoDB는 REPEATABLE READ에서도 gap lock으로 대부분 방지함

---

## 한 줄 요약

> 격리 수준 = Read View를 언제 만드느냐. READ COMMITTED는 쿼리마다, REPEATABLE READ는 트랜잭션 시작 시 한 번.

참고: mvcc.md, read-view.md, undo-log.md
