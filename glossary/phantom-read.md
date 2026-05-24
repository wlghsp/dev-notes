# phantom-read

**"같은 범위 조건으로 쿼리를 두 번 실행했는데 row 수가 달라지는 현상"**

---

## 언제 발생하나

트랜잭션 B가 범위 SELECT를 실행한 뒤, 트랜잭션 A가 그 범위에 해당하는 row를 INSERT하고 커밋한다. 이후 B가 같은 범위 SELECT를 다시 실행하면 없던 row가 나타난다.

```
트랜잭션 B: SELECT * WHERE age > 20 → 3건
트랜잭션 A: INSERT (age=25), COMMIT
트랜잭션 B: SELECT * WHERE age > 20 → 4건
→ 처음엔 없던 row가 나타남 (유령처럼)
```

non-repeatable read는 기존 row의 값이 바뀌는 것이고, phantom read는 row 자체가 생기거나 사라지는 것이다.

---

## non-repeatable read와의 차이

non-repeatable read — 이미 존재하던 row의 값이 변경됨 (UPDATE)  
phantom read — 없던 row가 새로 생기거나 사라짐 (INSERT / DELETE)

같은 "읽기 일관성 깨짐"이지만 원인이 다르다. REPEATABLE READ의 Read View 고정은 값 변경은 막지만, 잠금 없이는 새 row 삽입을 완전히 막기 어렵다.

---

## InnoDB에서 phantom read가 잘 안 생기는 이유

REPEATABLE READ에서 이론상 발생 가능하지만, InnoDB는 gap lock으로 대부분 차단한다.

gap lock은 인덱스 범위 사이의 "빈 공간"에 잠금을 건다. 트랜잭션 B가 `age > 20` 범위를 읽으면, 그 범위에 해당하는 gap에 잠금이 걸려 트랜잭션 A가 그 범위에 INSERT하지 못한다.

단, gap lock이 걸리지 않는 상황(잠금 없는 일반 SELECT)에서는 phantom read가 발생할 수 있다.

---

## 어느 격리 수준에서 발생하나

REPEATABLE READ 이하에서 이론상 발생한다. SERIALIZABLE에서는 모든 SELECT에 shared lock이 걸려 완전히 차단된다.

InnoDB REPEATABLE READ는 gap lock 덕분에 사실상 대부분의 phantom read를 막는다.

---

## 한 줄 요약

> 범위 쿼리 결과에 없던 row가 끼어드는 현상. InnoDB는 gap lock으로 대부분 방어한다.

참고: isolation-level.md, read-view.md, mvcc.md
