# debezium-snapshot

**"처음 시작할 때 기존 데이터를 통째로 읽어오는 초기 적재 과정"**

binlog는 최근 변경만 담고 있다. Debezium이 처음 연결됐을 때 binlog만 읽으면, 이전부터 있던 데이터는 소비자가 알 방법이 없다. 스냅샷은 이 문제를 해결하기 위한 초기 적재다.

---

## 동작 방식

처음 시작 시 Debezium은 대상 테이블을 전체 SELECT한다. 읽은 각 row를 `op: r` 이벤트로 Kafka에 발행한다. 스냅샷이 끝나면 그 시점의 binlog position부터 이어서 읽는다.

```
스냅샷 시작
  → 테이블 전체 SELECT (op: r 이벤트 발행)
  → 스냅샷 완료 시점의 binlog position 기록
  → 이후 binlog 스트림 이어서 읽기
```

스냅샷 중에도 DB는 계속 변경될 수 있다. Debezium은 스냅샷 시작 시점의 consistent snapshot(트랜잭션 격리)을 사용해서 중간에 바뀐 데이터가 섞이지 않도록 한다.

---

## 스냅샷 모드 종류

- `initial` — 오프셋이 없을 때(처음 시작)만 스냅샷. 이후엔 binlog만 읽음 (기본값)
- `never` — 스냅샷 없이 binlog만. 기존 데이터가 이미 적재돼 있거나 필요 없을 때
- `always` — 매 시작마다 스냅샷을 다시 찍음. 매번 전체 재적재가 필요한 경우

대부분은 `initial`로 충분하다. `never`는 offset을 직접 지정해서 특정 binlog position부터 읽고 싶을 때 쓴다.

---

## 한 줄 요약

> 스냅샷 = binlog 이전의 기존 데이터를 초기에 한 번 통째로 읽어서 Kafka로 발행하는 과정.

참고: debezium.md
참고: debezium-event.md
참고: offset.md
