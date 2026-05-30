# PITR (Point-In-Time Recovery)

**"특정 시점의 DB 상태로 되돌리는 복구 기법"**

실수로 테이블을 DROP했거나, 잘못된 UPDATE가 커밋된 경우 — 그 직전 시점으로 DB를 복구할 수 있다. 풀 백업 하나로는 최신 데이터를 살릴 수 없으니, 변경 로그를 시간 순서대로 재실행하는 방식으로 원하는 시점을 재현한다.

---

## 복구 방식

1. 가장 가까운 과거 시점의 full dump를 복원한다
2. 그 dump 이후부터 복구 목표 시점 직전까지의 binlog를 순서대로 replay한다
3. 원하는 시점의 DB 상태가 재현된다

```bash
# full dump 복원
mysql -u root -p < dump_2026-05-29.sql

# binlog replay (특정 시간 범위)
mysqlbinlog \
  --start-datetime="2026-05-29 00:00:00" \
  --stop-datetime="2026-05-29 14:33:00" \
  /var/lib/mysql/mariadb-bin.000XXX | mysql -u root -p
```

dump 시점과 복구 목표 시점 사이의 binlog가 없으면 복구 불가능하다. binlog 보존 기간이 PITR 가능 범위를 결정한다.

---

## binlog와의 관계

PITR은 binlog에 의존한다. binlog가 "어떤 변경이 언제 일어났는가"를 순서대로 기록하기 때문에 시간 축 위에서 재생이 가능하다.

binlog가 없거나 중간에 끊겼으면 그 구간은 복구할 수 없다.

참고: binlog.md

---

## 복제와의 공통점

복제(Replication)와 PITR 모두 binlog에 의존한다는 점에서 같은 메커니즘을 공유한다.

- 복제 — binlog를 다른 서버로 전송해서 실시간으로 replay
- PITR — binlog를 시간 범위를 지정해서 수동으로 replay

방향만 다르다. 복제는 공간(다른 서버), PITR은 시간(과거 시점).

---

## 한 줄 요약

> PITR = full dump + binlog replay. 원하는 시점까지의 binlog가 있어야 가능하다.
