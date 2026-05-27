# non-repeatable-read

**"같은 트랜잭션 안에서 같은 쿼리를 두 번 실행했는데 결과가 다른 현상"**

---

## 언제 발생하나

트랜잭션 B가 첫 번째 SELECT를 실행한 뒤, 트랜잭션 A가 해당 row를 변경하고 커밋한다. 이후 B가 같은 SELECT를 다시 실행하면 다른 결과가 나온다.

```
트랜잭션 B: SELECT name → 'Lee'
트랜잭션 A: UPDATE name = 'Kim', COMMIT
트랜잭션 B: SELECT name → 'Kim'
→ 같은 트랜잭션인데 결과가 바뀜
```

첫 번째와 두 번째 읽기 사이에 다른 트랜잭션의 커밋이 끼어든 것이 원인이다.

---

## dirty read와의 차이

dirty read는 커밋되지 않은 변경을 읽는 것이고, non-repeatable read는 커밋된 변경을 읽는 것이다. dirty read보다 덜 위험하지만, 같은 트랜잭션 내에서 일관된 데이터를 기대하는 로직에서는 문제가 생긴다.

---

## 어느 격리 수준에서 발생하나

READ COMMITTED까지 발생한다. 이 수준은 쿼리를 실행할 때마다 Read View를 새로 만들기 때문에, 그 사이에 커밋된 변경이 반영된다.

REPEATABLE READ부터는 트랜잭션 시작 시 Read View를 한 번만 만들고 고정하기 때문에, 이후 커밋이 일어나도 같은 결과를 반환한다.

---

## 한 줄 요약

> 같은 쿼리를 두 번 실행했는데 결과가 다르다면, Read View가 고정되지 않은 것이다.

참고: isolation-level.md, read-view.md, mvcc.md
