# Buffer (버퍼)

**한 줄 정의**: 속도가 다른 두 주체 사이에서 데이터를 임시로 들고 있는 공간.

---

## 왜 필요한가?

빠른 쪽이 느린 쪽을 매번 직접 기다리면 낭비다.

```
CPU 속도:  ~1 ns
Disk 속도: ~10 ms
→ CPU가 Disk를 기다리면 99.9999% 시간을 놀게 됨
```

버퍼를 두면 느린 쪽 접근 횟수를 줄이고, 빠른 쪽은 버퍼에서 빠르게 가져간다.

---


## Buffer vs Cache

둘 다 "중간에 두는 임시 공간"이지만 방향이 다르다.

**Buffer**는 쓰기 대기가 목적이다. 빠른 쪽에서 생산한 데이터를 모아뒀다가 나중에 느린 쪽에 전달한다. 방향은 빠른 쪽 → 느린 쪽.

**Cache**는 읽기 재사용이 목적이다. 느린 쪽에서 한 번 읽어온 데이터를 빠른 쪽에 들고 있다가 같은 요청이 오면 다시 느린 쪽을 거치지 않는다. 방향은 느린 쪽 → 빠른 쪽.

실제로는 하나의 공간이 두 역할을 동시에 하기도 한다. MySQL Buffer Pool은 이름은 Buffer지만, Disk Page를 메모리에 들고 있는 Cache 역할도 함께 한다.

---

## Write-behind (Dirty Buffer)

버퍼에 쓴다고 해서 즉시 Disk에 반영되지 않는다. 변경된 상태를 **Dirty**라고 부르고, 실제로 Disk에 내려쓰는 시점을 **Flush**라고 한다.

```
App → Buffer에 쓰기 (Dirty 상태)
             ↓  (나중에 Flush)
           Disk
```

Flush 타이밍은 시스템마다 다르다.

- **MySQL InnoDB**: Buffer Pool이 꽉 차거나, checkpoint 주기에 도달하면 Dirty Page를 Flush한다.
- **Kafka Producer**: `linger.ms`가 경과하거나 `batch.size`를 초과하면 메시지 배치를 브로커로 전송한다.
- **OS Page Cache**: `dirty_expire_centisecs` 주기마다 또는 앱이 `fsync()`를 직접 호출하면 Flush한다.

원리는 동일하다 — **모아서 한 번에 내려써서 I/O 횟수를 줄인다.**

---

## 버퍼의 위험: 크래시 시 데이터 유실

버퍼는 메모리에 있기 때문에 크래시가 나면 Flush되지 않은 데이터는 사라진다.

```
크래시 발생
    ↓
Buffer Pool / Send Buffer 증발
    ↓
Flush 안 된 데이터 유실
```

각 시스템은 이 문제를 다르게 보완한다.

- **MySQL InnoDB**: Redo Log(WAL)를 사용한다. Flush 전에 변경 내용을 먼저 Disk의 로그에 기록하고, 크래시 후 재시작 시 로그를 재적용해서 복구한다.
- **Kafka**: `acks=all`과 Replication을 사용한다. 브로커 여러 대에 복제가 완료된 후에야 ack를 반환해서 유실을 막는다.
- **OS**: 앱이 `fsync()`를 명시적으로 호출해서 "지금 당장 Disk에 써라"고 강제할 수 있다.

버퍼 크기를 크게 잡을수록 성능은 올라가지만, 크래시 시 유실 가능 범위도 커진다. 이 트레이드오프는 어느 시스템에나 존재한다.

---

## Back-pressure: 버퍼가 꽉 찼을 때

버퍼는 무한하지 않다. 느린 쪽이 계속 밀리면 버퍼가 가득 차고, 그때 빠른 쪽을 어떻게 다룰지 결정해야 한다.

```
빠른 쪽 → [버퍼 FULL] → 느린 쪽이 아직 처리 중
                ↓
   어떻게 할 것인가?
   1. Block  — 빠른 쪽을 멈춰서 기다리게
   2. Drop   — 새로 오는 데이터를 버림
   3. 신호   — 빠른 쪽에 "천천히 보내라"고 알림
```

각 시스템의 선택은 다르다.

- **Kafka Producer**: `max.block.ms` 동안 Block하고, 시간이 초과되면 예외를 던진다.
- **TCP**: 슬라이딩 윈도우를 축소해서 수신 측이 송신 측의 속도를 직접 제어한다.
- **JVM G1GC TLAB**: TLAB이 소진되면 공유 힙에서 직접 할당하거나 GC를 트리거한다.

Back-pressure는 버퍼 포화 상태를 상류(빠른 쪽)에 전달하는 메커니즘이다. 이게 없으면 버퍼 포화 → 데이터 유실 또는 OOM으로 이어진다.

---

## DB에서의 버퍼: Buffer Pool

MySQL InnoDB의 Buffer Pool은 이 개념의 대표적인 구현체다.

```
애플리케이션
     ↓
Storage Engine
     ↓
Buffer Pool (메모리) ← 여기서 찾으면 Hit, 없으면 Miss
     ↓ Miss
  .ibd 파일 (디스크)
```

- **Hit**: 메모리에서 바로 반환 — 디스크 접근 없음
- **Miss**: 디스크에서 Page(16KB)를 읽어 Buffer Pool에 올린 후 반환

버퍼가 클수록 Hit율이 올라가고 I/O가 줄어든다. 운영 환경에서 가용 메모리의 70~80%를 Buffer Pool에 할당하는 이유가 이것이다.

```sql
-- Buffer Pool 크기 확인
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Hit율 확인 (BUFFER POOL AND MEMORY 섹션)
SHOW ENGINE INNODB STATUS\G
```

## 관련 개념
- [DB Buffer Pool 상세](../db-internals/db-storage-basics.md#4-buffer-pool--디스크를-덜-읽기-위한-핵심-장치)
