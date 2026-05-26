# Galera 멀티센터 PK 충돌 방지

## 문제

양방향 CDC(주센터 ↔ DR센터)에서 각 센터가 독립적으로 AUTO_INCREMENT PK를 생성하면 동일한 id 값이 양쪽에서 만들어질 수 있다.

```
주센터: INSERT → id = 1
DR센터: INSERT → id = 1  (다른 데이터)
```

이 상태에서 CDC로 동기화하면 나중에 도착한 쪽이 먼저 온 걸 덮어쓴다. 데이터 유실이다.

---

## Galera의 기본 AUTO_INCREMENT 동작

Galera는 기본적으로 `wsrep_auto_increment_control = ON`이다. 클러스터 노드 수에 맞춰 자동으로 increment와 offset을 설정한다.

3노드 클러스터라면:
- node1: 1, 4, 7 ...
- node2: 2, 5, 8 ...
- node3: 3, 6, 9 ...

클러스터 **내부** 충돌은 막아준다. 하지만 **센터 간** 충돌은 막지 못한다. 주센터 node1과 DR센터 node1이 같은 id를 만든다.

---

## 해결: AUTO_INCREMENT Offset 분리

센터별로 생성되는 id 범위가 겹치지 않도록 설정한다.

```
주센터: offset=1, increment=2 → 1, 3, 5, 7 ... (홀수)
DR센터: offset=2, increment=2 → 2, 4, 6, 8 ... (짝수)
```

**주센터 my.cnf:**
```ini
wsrep_auto_increment_control = OFF
auto_increment_increment     = 2
auto_increment_offset        = 1
```

**DR센터 my.cnf:**
```ini
wsrep_auto_increment_control = OFF
auto_increment_increment     = 2
auto_increment_offset        = 2
```

`wsrep_auto_increment_control = OFF`가 핵심이다. 이게 ON이면 Galera가 노드 수 기준으로 수동 설정을 덮어쓴다.

---

## DR 전환 시나리오

평상시엔 주센터만 쓰기를 한다. id는 홀수만 생성된다.

주센터 장애로 DR센터가 쓰기를 받기 시작하면 id는 짝수만 생성된다. 주센터가 복구되어 재합류해도 offset이 다르기 때문에 PK 충돌이 없다.

---

## 주의사항

**기존 데이터가 있는 경우** — offset 분리 전에 이미 데이터가 양쪽에 있으면 id가 겹칠 수 있다. 현재 최대 id를 확인하고 DR센터 AUTO_INCREMENT 시작값을 조정해야 한다.

**센터가 늘면 increment도 바꿔야 한다** — 3센터가 되면 increment=3으로 바꾸고 기존 데이터를 조정해야 한다. 확장성 면에서 UUID PK가 근본적으로 더 안전하다.

**노드 재시작은 반드시 1노드씩** — 동시 재시작하면 quorum을 잃는다. 재시작 후 `wsrep_local_state_comment`가 `Synced`가 된 것을 확인하고 다음 노드로 넘어가야 한다.

---

## 검증

```sql
-- 설정 확인
SHOW VARIABLES LIKE 'auto_increment%';
SHOW VARIABLES LIKE 'wsrep_auto_increment_control';

-- 주센터: 1, 3, 5 순서로 생성되는지 확인
INSERT INTO test_table (name) VALUES ('a'), ('b'), ('c');
SELECT id FROM test_table ORDER BY id DESC LIMIT 3;
```
