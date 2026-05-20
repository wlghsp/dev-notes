# Galera Cluster 내부 동작 원리

> 구축은 해봤지만 "왜 이렇게 동작하는가"를 이해하기 위한 학습 문서입니다.
> 타이핑하면서 읽으면 더 잘 기억됩니다.

---

## 1. Galera는 일반 복제(Replication)와 무엇이 다른가

### 일반 MySQL/MariaDB 복제 (비교 기준)

일반 복제는 **Master → Slave** 구조입니다.

- Master에서 쓰기 발생 → binlog에 기록
- Slave가 binlog를 읽어 재실행 (비동기)
- **Slave는 항상 조금 뒤처져 있음** (replication lag)
- Slave에 직접 쓰기 불가 (read-only)
- Master가 죽으면 수동으로 failover 해야 함

### Galera의 접근 방식

Galera는 **Active-Active** 구조입니다.

- 모든 노드에 쓰기 가능
- 모든 노드가 항상 동일한 데이터 (사실상 동기 복제)
- 노드 1개가 죽어도 나머지가 자동으로 계속 서비스
- binlog 방식이 아닌 **wsrep (Write-Set Replication)** 사용

핵심 차이: 일반 복제는 "SQL을 다시 실행"하지만, Galera는 "변경된 행(row) 데이터 자체"를 전파합니다.

---

## 2. wsrep와 Write-Set

### wsrep란?

wsrep는 MySQL(InnoDB)과 Galera 복제 레이어 사이의 **추상화 API**입니다.

```
MySQL (InnoDB) → wsrep API → Galera Library → 네트워크 → 다른 노드들
```

MySQL은 wsrep를 직접 구현하지 않습니다. Galera Library가 wsrep provider로서 붙어서 동작합니다. 트랜잭션이 커밋될 때 InnoDB가 wsrep API를 호출하고, Galera가 그 변경 내용을 Write-Set으로 만들어 클러스터 전체에 전파합니다.

동작 흐름:

1. 트랜잭션 커밋 시도 → InnoDB가 wsrep API 호출
2. Galera가 Write-Set 생성 → 다른 노드에 브로드캐스트
3. 각 노드가 수신·검증 → 충돌 없으면 OK
4. 모든 노드가 동일한 순서로 커밋 (seqno로 순서 보장)

### Write-Set이란?

Galera에서 트랜잭션이 복제되는 단위를 **Write-Set**이라고 합니다.

### Write-Set의 구성

트랜잭션 하나가 커밋될 때 Galera는 다음을 묶어서 Write-Set을 만듭니다:

```
Write-Set = {
    변경된 행들의 before/after 이미지,
    트랜잭션이 읽은 행들의 key (충돌 감지용),
    GTID (글로벌 트랜잭션 ID)
}
```

### Write-Set 전파 흐름

```
node1에서 UPDATE 실행 (로컬에서 트랜잭션 진행 중)
    ↓
COMMIT 호출
    ↓
Write-Set 확정 (변경된 행, 키, GTID 묶음)
    ↓
모든 노드에 브로드캐스트 (4567 포트, TCP/UDP)
    ↓
각 노드에서 Certification (충돌 검사)
    ↓
충돌 없으면 → 모든 노드 커밋
충돌 있으면 → 해당 트랜잭션 롤백
```

중요한 점: **node1에서 커밋된 것처럼 보여도 다른 노드의 승인을 받기 전까지는 진짜 커밋이 아닙니다.**

---

## 3. Certification-Based Replication (인증 기반 복제)

Galera의 핵심 개념입니다. "어떻게 충돌을 감지하는가"의 답입니다.

### 충돌이란?

두 노드에서 같은 행을 거의 동시에 수정하면 충돌(conflict)이 발생합니다.

```
node1: UPDATE users SET name='Alice' WHERE id=1;  (동시에)
node2: UPDATE users SET name='Bob'   WHERE id=1;
```

어떤 것이 이겨야 할까요?

### Certification 동작 방식

각 노드는 Write-Set을 받으면 다음을 검사합니다:

1. 이 Write-Set이 수정한 행의 key를 확인
2. 내가 최근에 커밋한 트랜잭션 중 같은 key를 건드린 게 있는가?
3. 있다면 → **충돌 → 나중에 온 트랜잭션 롤백**
4. 없다면 → **인증 통과 → 커밋**

### 결과

- 충돌은 자동 감지되고 자동 롤백됨
- 애플리케이션은 `deadlock` 에러를 받으면 재시도해야 함
- Galera가 "사실상 동기"인 이유: 모든 노드가 같은 Certification 결과를 가짐

---

## 4. Commit 순서 — 로컬 커밋 vs 클러스터 커밋

일반 DB와 Galera의 커밋 흐름 비교입니다.

### 일반 MariaDB 커밋

```
COMMIT 명령
    → Redo Log (WAL) fsync → 완료 응답
    → (이후 background) Buffer Pool dirty page → 데이터 파일 flush
```
`   
COMMIT 시점에 디스크에 기록되는 건 Redo Log입니다. 실제 데이터 파일은 Buffer Pool의 dirty page가 나중에 flush될 때 반영됩니다.

### Galera 커밋 (2-phase commit과 유사)

```
COMMIT 명령
    → Write-Set 생성
    → 클러스터 전체에 브로드캐스트
    → 모든 노드에서 Certification
    → 전체 OK → 로컬 Redo Log fsync + 다른 노드에 Apply
    → 클라이언트에 응답 반환
    → (이후 background) 각 노드 Buffer Pool dirty page → 데이터 파일 flush
```

이 때문에 Galera는 일반 MariaDB보다 **쓰기 지연(latency)이 높습니다.**
네트워크 왕복 시간(RTT)이 커밋 시간에 직접 영향을 줍니다.

> 같은 IDC 내: 수 ms 추가
> 원거리(센터 간): 수십~수백 ms 추가 → 주요 병목

---

## 5. Quorum — 왜 3노드인가

### Quorum이란?

클러스터가 "정상적으로 동작할 수 있는 상태"를 판단하는 기준입니다.

```
Quorum = 전체 노드의 과반수 이상이 살아있어야 함
```

### 노드 수별 비교

| 노드 수 | Quorum 기준 | 허용 장애 노드 |
|---------|------------|--------------|
| 1       | 1          | 0            |
| 2       | 2          | 0 ← 의미없음  |
| 3       | 2          | 1            |
| 4       | 3          | 1            |
| 5       | 3          | 2            |

**2노드가 안 되는 이유**: 노드 1개가 죽으면 남은 1개가 과반수인지 알 수 없습니다.
네트워크 단절인지, 상대가 죽은 건지 구분이 불가능합니다.
이 상태에서 계속 쓰기를 허용하면 **split-brain**이 발생합니다.

### Split-brain이란?

```
[node1] ← 네트워크 단절 → [node2]

node1: "나만 살아있다. 내가 Primary다." → 계속 쓰기
node2: "나만 살아있다. 내가 Primary다." → 계속 쓰기

→ 두 노드의 데이터가 달라짐 → 나중에 합칠 수 없음
```

3노드면 네트워크가 분리되어도 한쪽은 반드시 2개 이상 → Quorum 확보 → 정상 운영.
나머지 1개는 Quorum 없음 → 쓰기 거부 → 데이터 보호.

---

## 6. SST vs IST — 언제 전체 복사, 언제 증분 복사

노드가 클러스터에 새로 참여하거나 오랫동안 떠나 있다가 돌아올 때 데이터를 맞추는 방법입니다.

### SST (State Snapshot Transfer) — 전체 복사

```
새 노드 또는 데이터가 너무 뒤처진 노드에 적용

Donor 노드 → 전체 데이터 복사 → Joiner 노드
```

- rsync, mysqldump, xtrabackup 방식 사용 가능
- **rsync 방식**: Donor 노드가 SST 중 쿼리를 처리하지 못함 (중단)
- **xtrabackup 방식**: Donor 노드가 서비스 유지하며 복사 가능 (권장)
- 데이터가 많을수록 오래 걸림

### IST (Incremental State Transfer) — 증분 복사

```
잠깐 떠났다가 돌아온 노드에 적용 (gcache에 변경분이 남아있을 때)

Donor 노드 → 빠진 동안의 Write-Set만 전송 → Joiner 노드
```

- Galera는 gcache라는 내부 버퍼에 최근 Write-Set을 보관
- 빠진 시간이 짧으면 gcache에서 찾아 IST로 처리 → 빠름
- gcache에 없으면 → SST로 전환 (전체 복사)

### 언제 어떤 게 일어나는가

```
노드 재시작 (잠깐 중단)  → IST 시도 → gcache 있으면 IST, 없으면 SST
노드 신규 추가           → SST (데이터 없음)
노드 오래 중단 후 복귀   → SST (gcache 만료)
```

운영 시 주의: `gcache.size`를 충분히 크게 설정하면 IST 성공 확률이 높아집니다.

---

## 7. Flow Control — 느린 노드가 있을 때

### 문제 상황

node1, node2는 빠른데 node3이 느리면?

```
Write-Set이 계속 쌓임 → node3의 수신 큐(recv_queue)가 증가
→ node3이 점점 뒤처짐 → 결국 클러스터 전체 성능 저하
```

### Flow Control 동작

Galera는 느린 노드를 감지하면 클러스터 전체의 쓰기 속도를 늦춥니다.

```
node3의 recv_queue가 임계값 초과
    ↓
node3이 FC_PAUSE 신호 브로드캐스트
    ↓
모든 노드가 Write-Set 전송 일시 중지
    ↓
node3이 따라잡음
    ↓
FC_CONTINUE → 정상 재개
```

### 모니터링 방법

```sql
SHOW STATUS LIKE 'wsrep_flow_control_paused';
-- 0에 가까울수록 좋음, 1에 가까우면 Flow Control이 자주 발생
-- 0.1 이상이면 병목 노드가 있다는 신호

SHOW STATUS LIKE 'wsrep_local_recv_queue_avg';
-- 평균 수신 대기 큐 크기, 0이 이상적
```

Flow Control이 자주 발생하면 → 느린 노드의 디스크/CPU 확인이 필요합니다.

---

## 8. grastate.dat의 의미

클러스터 재시작 시 문제가 생기는 이유가 이 파일에 있습니다.

### 파일 내용

```
# GALERA saved state
version: 2.1
uuid:    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   ← 클러스터 식별자
seqno:   145                                    ← 마지막으로 처리한 트랜잭션 번호
safe_to_bootstrap: 0                            ← 이 노드로 부트스트랩해도 안전한가
```

### seqno

- 각 Write-Set에 순서대로 부여되는 번호
- 노드가 정상 종료되면 마지막 seqno가 기록됨
- 비정상 종료(전원 차단 등)되면 `-1`로 기록됨 → "모름" 상태

### safe_to_bootstrap

- 클러스터 전체가 내려갔을 때, 어느 노드부터 시작해야 하는가의 답
- 정상 종료된 마지막 노드에 `1`이 자동으로 설정됨
- 이 값이 `1`인 노드부터 `galera_new_cluster`를 실행해야 함
- 모두 `-1`이면 (비정상 종료) → 수동으로 가장 최신 노드를 선택해 `1`로 설정

### 전체 동시 재시작 시 왜 문제가 되는가

```
정상 순차 종료:
node3 중지 → node2 중지 → node1 중지 (마지막)
→ node1의 safe_to_bootstrap = 1 자동 설정
→ node1으로 부트스트랩 가능

동시 재시작 (문제):
node1, node2, node3 동시 중지
→ 어느 노드가 마지막인지 알 수 없음
→ 모두 safe_to_bootstrap = 0, seqno = -1
→ 수동 복구 필요
```

---

## 9. 핵심 포트 정리

| 포트 | 용도 | 설명 |
|------|------|------|
| 3306 | MySQL | 일반 DB 접속 |
| 4567 | Galera replication | Write-Set 브로드캐스트 (TCP+UDP) |
| 4568 | IST | 증분 데이터 전송 |
| 4444 | SST | 전체 스냅샷 전송 |

---

## 10. 요약 — 한 줄씩 외우기

- **Write-Set**: 트랜잭션이 바꾼 행 데이터 묶음. SQL이 아닌 데이터 자체를 전파
- **Certification**: 충돌 감지 메커니즘. 같은 행을 동시에 수정하면 나중 것이 롤백
- **Quorum**: 과반수 이상 살아있어야 쓰기 허용. 3노드면 1개 장애 허용
- **Split-brain**: Quorum 없이 각자 쓰기하면 데이터가 갈라짐. Galera는 이를 방지
- **SST**: 전체 복사. Donor 노드 부하 큼 (rsync는 서비스 중단)
- **IST**: 증분 복사. gcache에 변경분이 있을 때만 가능. 빠름
- **Flow Control**: 느린 노드가 있으면 클러스터 전체를 늦춤. recv_queue로 감지
- **seqno**: 트랜잭션 순서 번호. -1이면 비정상 종료
- **safe_to_bootstrap**: 부트스트랩 가능 여부. 전체 다운 복구 시 핵심

---

## 11. Galera를 선택해야 할 때

Galera가 강력하다고 해서 항상 정답은 아닙니다. 현업에서는 일반 복제가 여전히 더 많이 쓰입니다.

일반 복제로 충분한 경우:

- **읽기 분산이 목적**일 때 — Master 1대 + Replica 여러 대로 충분
- **쓰기 레이턴시가 중요**할 때 — Galera는 커밋마다 클러스터 승인을 기다림. RTT가 직접 영향을 줌
- **Analytics/배치 분리**가 목적일 때 — 무거운 쿼리를 Replica로 보내 Master 보호
- **원거리 IDC 복제**가 필요할 때 — Galera는 원거리에서 레이턴시 문제가 심함. 비동기 복제가 현실적

Galera가 진짜 필요한 경우:

- 어느 노드에나 쓰기가 가능해야 하는 **Active-Active** 구성
- 노드 장애 시 **자동 Failover** 없이는 안 되는 서비스
- 복제 lag 없이 모든 노드가 동일한 상태여야 하는 **강한 일관성** 요구

단순히 "고가용성이 필요하다"는 이유만으로 Galera를 선택하면 운영 복잡도만 올라갑니다. 일반 복제 + 수동 Failover로 충분한 경우가 많습니다.
