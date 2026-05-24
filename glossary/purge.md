# purge

**"더 이상 필요 없는 undo log와 delete-marked row를 실제로 삭제하는 백그라운드 작업"**

InnoDB는 DELETE나 UPDATE 시 데이터를 즉시 지우지 않는다. purge 스레드가 나중에 안전하다고 판단되면 그때 실제로 삭제한다.

---

## 왜 즉시 삭제하지 않나

MVCC 때문이다. DELETE된 row도 과거 시점을 읽어야 하는 트랜잭션에게는 여전히 필요할 수 있다. 그 트랜잭션이 끝나기 전에 지워버리면 일관된 읽기가 깨진다.

그래서 InnoDB는 두 단계로 나눈다.

1. DELETE 실행 시 → row에 delete mark만 표시, undo log 기록
2. purge 실행 시 → 해당 row를 참조하는 트랜잭션이 없으면 실제 삭제

---

## purge가 처리하는 것

**delete-marked row 제거**

DELETE된 row는 delete mark가 붙은 채로 남아있다. purge가 물리적으로 제거한다.

**undo log 정리**

커밋된 트랜잭션의 undo log도 참조하는 트랜잭션이 없어지면 purge가 삭제한다.

---

## long running transaction과의 관계

purge는 "현재 열려있는 가장 오래된 트랜잭션의 시작 시점" 이전 버전만 삭제할 수 있다. 그 트랜잭션이 혹시라도 과거 버전을 읽을 수 있기 때문이다.

오래된 트랜잭션이 계속 열려있으면 → purge가 그 이후 쌓인 undo log를 지우지 못함 → undo log가 계속 증가 → 디스크 사용량 증가, 성능 저하.

"DELETE했는데 디스크가 안 줄어든다"는 대부분 이 상황이다.

---

## 한 줄 요약

> purge = MVCC 안전성을 확인한 뒤 delete-marked row와 불필요한 undo log를 실제로 제거하는 백그라운드 작업.

참고: undo-log.md, mvcc.md
