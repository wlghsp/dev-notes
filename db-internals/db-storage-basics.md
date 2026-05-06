# Phase 0: 스토리지 기초 — 데이터는 디스크에 어떻게 저장되는가

> 완료 기준: "Page가 뭔지, Buffer Pool이 왜 필요한지 동료에게 5분 설명 가능"

---

## 왜 이걸 먼저 알아야 하나?

인덱스가 왜 빠른지, 트랜잭션이 왜 느려지는지, WAL이 왜 필요한지 — 이 질문들은 결국 하나로 수렴한다: 디스크에서 데이터를 어떻게 읽고 쓰는가.
여기서 안 쌓으면 이후 모든 개념이 암기가 된다.

---

## 1. Disk I/O가 왜 병목인가

```
CPU 연산:      ~1 ns
메모리 접근:   ~100 ns
SSD 읽기:      ~100 µs   (메모리보다 1,000배 느림)
HDD 읽기:      ~10 ms    (메모리보다 100,000배 느림)
```

DB의 데이터는 결국 디스크에 있다. 쿼리가 느린 이유의 대부분은 **CPU가 아니라 Disk I/O**다.

그래서 DB 설계의 핵심 목표는 하나다: **디스크를 최대한 적게, 효율적으로 읽는다.**

그 목표를 달성하기 위해 DB는 두 가지 전략을 쓴다 — 한 번 읽을 때 크게 묶어서 읽고(Page), 자주 쓰는 건 메모리에 캐시해둔다(Buffer Pool). 2번과 3번이 각각 그 이야기다.

---

## 2. Disk 구조 — HDD와 SSD의 동작 원리

디스크 I/O가 왜 느린지 이해하려면 하드웨어 수준에서 어떻게 동작하는지 알아야 한다.

### HDD (Hard Disk Drive)

HDD는 자성 원판(platter)이 회전하면서 데이터를 읽고 쓴다.

```
         ┌─────────────────┐
         │  Read/Write Head │  ← 헤드가 원판 표면을 따라 이동
         └────────┬────────┘
                  │
      ┌───────────▼──────────┐
      │   Platter (회전)      │
      │  ┌───┬───┬───┬───┐  │
      │  │   │   │   │   │  │  ← Track (동심원 레이어)
      │  └───┴───┴───┴───┘  │
      │      ↑               │
      │   Sector: 512B~4KB  │  ← 실제 읽기/쓰기의 최소 단위
      └──────────────────────┘
```

HDD I/O 비용의 구성:
- **Seek time**: 헤드가 목표 트랙으로 이동하는 시간 — I/O 비용의 대부분
- **Rotational latency**: 목표 섹터가 헤드 아래로 회전해 오는 시간
- **Transfer time**: 실제 데이터 전송 시간 — 위 두 비용에 비하면 미미함

결론: HDD에서 Random I/O가 느린 이유는 **매 I/O마다 헤드 이동(seek)**이 발생하기 때문이다. 순차적으로 읽으면 헤드 이동 없이 연속해서 읽을 수 있어 훨씬 빠르다.

### SSD (Solid State Drive)

SSD는 물리적 이동 부품이 없다. 대신 전기적 신호로 플래시 메모리 셀에 데이터를 저장한다.

```
SSD 계층 구조 (작은 것 → 큰 것):
  Memory Cell → String → Array → Page → Block → Plane → Die
                                  ↑           ↑
                             읽기/쓰기 단위  삭제 단위
                             (보통 4~16KB)  (수백KB ~ 수MB)
```

SSD의 핵심 제약:
- **읽기/쓰기**: Page 단위 (4~16KB)
- **삭제**: Block 단위 — Page보다 훨씬 큰 단위로만 삭제 가능
- 이 불일치가 **Write Amplification**을 유발한다 — 1바이트를 바꾸려면 Block 전체를 읽어서 새 Block에 다시 쓴 뒤 기존 Block을 지워야 함

SSD는 seek time이 없어서 HDD보다 Random I/O가 훨씬 빠르지만, 여전히 메모리보다는 1,000배 이상 느리다. DB가 Buffer Pool을 유지하는 이유는 SSD 환경에서도 여전히 유효하다.

---

## 3. Page — DB의 기본 I/O 단위

디스크 I/O의 핵심 특성이 하나 있다: **"얼마나 많이 읽냐"가 아니라 "몇 번 읽냐"가 비용이다.**

HDD는 헤드가 해당 위치로 이동하는 seek time이 I/O 비용의 대부분이다. 실제 데이터 전송 시간은 그것에 비하면 미미하다. SSD도 I/O 요청 자체의 레이턴시가 지배적이다. 결론: **4KB를 읽든 16KB를 읽든 I/O 1회 비용은 거의 같다.**

DB는 이 특성을 이용한다. 어차피 비용이 같다면, 한 번 읽을 때 더 많이 묶어서 읽어오는 게 이득이다. 그래서 DB는 자체적인 단위인 **Page**를 정의하고, 항상 Page 단위로 읽고 쓴다.

```
┌─────────────────────────────────────────┐
│              Disk Block                 │  ← OS/하드웨어 단위 (보통 512B ~ 4KB)
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│               DB Page                   │  ← DB가 정의한 단위 (InnoDB 기본: 16KB)
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  │
│  │ row  │ │ row  │ │ row  │ │ row  │  │
│  └──────┘ └──────┘ └──────┘ └──────┘  │
└─────────────────────────────────────────┘
```

**왜 Page 크기를 16KB로 정했는가?**

어차피 I/O 1회 비용이 같다면, 한 번 읽을 때 더 많이 읽어오는 게 이득이다. 하지만 무작정 크게 하면 안 된다.

- **너무 작으면 (4KB)**: row 하나 읽었는데 근처 row 조회 시 또 I/O 발생
- **너무 크면 (1MB)**: row 1개 때문에 1MB를 Buffer Pool(메모리)에 올려야 함 → 메모리 낭비
- **16KB (InnoDB 기본)**: row 여러 개를 한 번에 올리고, 근처 row 재사용 — I/O 최소화 + 메모리 효율 균형

16KB는 1990년대 말 InnoDB 설계 당시 서버 메모리(256MB~1GB)와 일반적인 row 크기(수백 바이트)를 기준으로 실험적으로 정착된 값이다. PostgreSQL은 8KB, SQL Server는 8KB로 DB마다 다르지만, 이후 메모리가 충분히 저렴해지면서 InnoDB의 16KB가 균형점으로 검증됐다.

**핵심 정리:**
- Disk Block은 하드웨어가 강제하는 최소 단위 (DB가 선택할 수 없음)
- DB Page는 그 위에서 DB가 효율을 위해 선택한 단위 (I/O 횟수를 줄이기 위한 전략)
- 인덱스 구조(B-Tree)도 Page 단위로 노드를 구성한다 — B-Tree가 Page와 어떻게 맞물리는지는 Phase 2에서 다루지만, 지금은 "Page 하나 = 인덱스 트리의 노드 하나"라는 것만 기억해두면 된다

MySQL InnoDB의 기본 Page 크기는 **16KB**다.

```sql
SHOW VARIABLES LIKE 'innodb_page_size';
```
> 📷 **[직접 실행 결과 캡쳐 첨부]**

---

## 4. Data Files와 Index Files

DB는 데이터를 어떻게 파일에 담는가? 크게 두 종류의 파일이 있다.

**Data Files** — 실제 레코드를 저장하는 파일이다. 구성 방식에 따라 세 가지로 나뉜다.

1. **Heap-organized tables (Heap files)**
   - 레코드를 별도 순서 없이 그냥 쌓는다 (heap = "쌓아둔 더미")
   - 삽입은 빠르지만, 특정 레코드를 찾으려면 전체를 순회해야 해서 인덱스 없이는 느리다
   - PostgreSQL의 기본 방식

2. **Hash-organized tables**
   - 레코드를 특정 컬럼의 해시값으로 분류해서 저장
   - 특정 키로 exact match 조회는 빠르지만, 범위 조회가 불가능

3. **Index-organized tables (IOT)**
   - 데이터 자체가 인덱스 순서로 정렬되어 저장 — 즉, 테이블이 인덱스 그 자체
   - InnoDB의 방식: Primary Key 순서로 B-Tree 구조에 데이터가 들어있다
   - Primary Key로 조회 시 인덱스 탐색이 곧 데이터 접근이라 효율적

**Index Files** — 데이터를 빠르게 찾기 위한 별도 파일이다.

InnoDB에서 Secondary Index(Primary Key 외 인덱스)를 만들면, 해당 인덱스 파일에는 **인덱스 컬럼 값 + Primary Key 값**만 저장된다. 실제 레코드 전체를 복사해 두지 않는다.

이 구조 때문에 Secondary Index 조회는 두 단계를 거친다:

```
Secondary Index 조회 (예: WHERE email = 'jiho@...')

1단계: Secondary Index(email) B-Tree 탐색
       → email 컬럼 값 + Primary Key(id) 값 획득

2단계: Primary Index(id) B-Tree 탐색
       → 실제 레코드 반환

(= 2번의 B-Tree 탐색)
```

즉, Secondary Index는 레코드를 직접 가리키지 않고 Primary Key를 거쳐서 도달한다.

만약 Secondary Index가 레코드의 물리적 위치(디스크 주소)를 직접 저장했다면, 페이지 분할로 레코드가 다른 위치로 이동할 때마다 그 레코드를 가리키는 모든 Secondary Index를 찾아서 주소를 바꿔야 한다. InnoDB는 Secondary Index에 Primary Key 값을 저장하기 때문에, 레코드가 이동해도 Primary Index만 업데이트하면 된다. Secondary Index는 손댈 필요가 없다.

Covering Index를 만들면 이 2단계를 건너뛸 수 있어서 빠르다.

### 삭제는 즉시 지우지 않는다 — Deletion Markers (Tombstones)

레코드를 DELETE할 때, DB는 해당 레코드를 파일에서 즉시 제거하지 않는다. 대신 **삭제됐다는 표시(deletion marker / tombstone)**만 남긴다.

왜 이렇게 하는가? 이유가 여러 겹이다:
- 디스크에서 파일 중간의 데이터를 즉시 제거하고 구멍을 메우는 작업은 비용이 크다
- MVCC 환경에서 다른 트랜잭션이 아직 해당 레코드를 읽고 있을 수 있다
- LSM-Tree 기반 엔진(RocksDB 등)은 tombstone을 명시적으로 SSTable에 기록해두고, 이후 Compaction 시점에 실제로 제거한다

InnoDB에서는 Undo Log가 이 역할을 담당한다. 삭제된 레코드는 Undo Log에 이전 버전이 남아있고, 다른 트랜잭션의 Read가 모두 끝나면 Purge 스레드가 실제로 정리한다.

---

## 5. Random I/O vs Sequential I/O

디스크 I/O에는 두 종류가 있고, 성능 차이가 크다.

```
Sequential I/O (순차 읽기)
┌──┬──┬──┬──┬──┬──┬──┬──┐
│P1│P2│P3│P4│P5│P6│P7│P8│  ← 연속된 Page를 순서대로 읽음
└──┴──┴──┴──┴──┴──┴──┴──┘
   →→→→→→→→→→→→→→→→

Random I/O (랜덤 읽기)
┌──┬──┬──┬──┬──┬──┬──┬──┐
│P1│  │P3│  │  │P6│  │P8│  ← 흩어진 Page를 각각 읽음
└──┴──┴──┴──┴──┴──┴──┴──┘
   ↗        ↗     ↗    ↗   (헤드 이동 비용 발생 — HDD에서 특히 심각)
```

- **Full Table Scan**: Sequential I/O → 데이터가 많아도 예측 가능한 속도
- **Index Scan (비효율적)**: Random I/O → 인덱스가 있어도 느릴 수 있음

**"인덱스가 있는데 왜 Full Scan이 더 빠르지?"**

이게 처음엔 직관에 반한다. 이유는 **Selectivity(선택도)** 개념으로 설명된다.

Selectivity = 인덱스 조건이 전체 row 중 얼마나 걸러내는가.

```
Selectivity 높음 (인덱스가 유리한 경우)
  SELECT * FROM orders WHERE order_id = 9999;
  → 1억 건 중 1건만 반환 → Random I/O 1회 → 인덱스가 압도적으로 유리

Selectivity 낮음 (Full Scan이 유리한 경우)
  SELECT * FROM orders WHERE status = 'PENDING';
  → 1억 건 중 3,000만 건 반환
  → 인덱스를 타면: 3,000만 번 Random I/O (각각 다른 Page로 점프)
  → Full Scan이면: Page를 처음부터 끝까지 Sequential I/O 1회
  → Full Scan이 오히려 빠름
```

MySQL 옵티마이저는 이 판단을 통계 기반으로 자동으로 한다. `EXPLAIN`에서 `type: ALL`(Full Scan)이 나올 때 무조건 나쁜 게 아니라, Selectivity가 낮아서 옵티마이저가 의도적으로 선택한 경우일 수 있다.

---

## 6. Buffer Pool — 디스크를 덜 읽기 위한 핵심 장치

매번 디스크에서 Page를 읽으면 느리다. 그래서 DB는 **메모리에 Page 캐시**를 둔다. 이게 **Buffer Pool**이다.

```mermaid
flowchart TD
    Q["쿼리 실행</br>(Page 필요)"]
    BP["Buffer Pool</br>(메모리 캐시)"]
    DISK["Disk</br>(.ibd 파일)"]

    Q -->|Page 요청| BP
    BP -->|Hit: 캐시에 있음| Q
    BP -->|Miss: 캐시에 없음| DISK
    DISK -->|Page 로드| BP
    BP -->|Page 반환| Q

    style BP fill:#4a90d9,color:#fff
    style DISK fill:#e67e22,color:#fff
```

**Buffer Pool Hit** — 디스크 접근 없이 메모리에서 바로 반환. 빠름.
**Buffer Pool Miss** — 디스크에서 Page를 읽어서 Buffer Pool에 올린 후 반환. 느림.

따라서 **Buffer Pool 크기 = DB 성능에 직접적인 영향**.

MySQL InnoDB 기본값은 128MB다. 실제 운영 환경에서는 가용 메모리의 **70~80%** 를 잡는 게 일반적인 권장값이다.

왜 70~80%인가? 100%를 주면 안 되는 이유가 있다:

```
서버 메모리 = Buffer Pool + 나머지
                              ├── OS 커널
                              ├── MySQL 프로세스 자체 (연결 관리, 정렬 버퍼 등)
                              ├── Redo Log 버퍼
                              └── 기타 스레드 스택

→ Buffer Pool에 100% 주면 OS와 MySQL 자체가 메모리 부족으로 죽음
→ 70~80%는 "Buffer Pool에 최대한 주되, 나머지가 굶지 않는" 경험적 균형점
```

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```
> 📷 **[직접 실행 결과 캡쳐 첨부]**

---

## 7. Page의 생애주기

```mermaid
sequenceDiagram
    participant Q as 쿼리
    participant BP as Buffer Pool
    participant D as Disk

    Q->>BP: Page 요청
    BP->>BP: 캐시 확인 (Hit?)

    alt Buffer Pool Hit
        BP-->>Q: Page 즉시 반환
    else Buffer Pool Miss
        BP->>D: Page 읽기 요청
        D-->>BP: Page 로드 (16KB)
        BP-->>Q: Page 반환
    end

    Q->>BP: Page 수정 (INSERT/UPDATE)
    BP->>BP: Dirty Page로 표시
    Note over BP: 즉시 디스크에 쓰지 않음

    BP->>D: Checkpoint 시점에 Flush
```

**Dirty Page**: 메모리에서 수정됐지만 아직 디스크에 반영되지 않은 Page.

DB는 수정이 발생해도 즉시 디스크에 쓰지 않는다. 왜냐하면:

```
row 1건 수정 → 즉시 디스크 쓰기
row 2건 수정 → 즉시 디스크 쓰기
row 3건 수정 → 즉시 디스크 쓰기
...
→ 수정 1번마다 Disk I/O 1번 → 너무 느림

대신:
수정 발생 → 메모리(Buffer Pool)에만 반영 → Dirty Page로 표시
수정 발생 → 메모리에만 반영 → Dirty Page
...
→ Checkpoint 시점에 Dirty Page 모아서 한 번에 디스크에 씀 → I/O 횟수 최소화
```

단, 여기서 문제가 생긴다. 메모리에만 있는 수정 내용이 서버가 갑자기 꺼지면 사라진다.
이걸 막기 위해 **Redo Log(WAL)** 가 존재한다 — 수정 내용을 디스크의 로그 파일에 먼저 순차적으로 기록해두고, 장애 시 이 로그로 복구한다. 이 구조가 Crash Recovery의 핵심이다 → Phase 4에서 상세히 다룬다.

---

## 8. 전체 그림

```mermaid
graph TD
    APP["애플리케이션<br/>(쿼리 요청)"]
    SE["Storage Engine<br/>(InnoDB)"]
    BP["Buffer Pool<br/>(메모리)"]
    LOG["Redo Log<br/>(WAL)"]
    DATA["Data Files<br/>(.ibd)"]

    APP --> SE
    SE --> BP
    BP -->|Miss| DATA
    SE -->|변경 발생| LOG
    BP -->|Flush| DATA

    style BP fill:#4a90d9,color:#fff
    style LOG fill:#27ae60,color:#fff
    style DATA fill:#e67e22,color:#fff
```

**Storage Engine**은 MySQL에서 실제 데이터를 읽고 쓰는 역할을 담당하는 레이어다. MySQL은 쿼리 파싱/실행 계획까지만 담당하고, 실제 디스크 I/O는 Storage Engine에 위임한다. InnoDB가 기본이며, 트랜잭션·Buffer Pool·Redo Log를 모두 Storage Engine이 관리한다.

**`.ibd` 파일**은 InnoDB가 테이블 데이터와 인덱스를 저장하는 실제 디스크 파일이다. 테이블마다 하나씩 생성된다(`테이블명.ibd`). 내부는 16KB Page의 연속으로 구성되어 있고, Buffer Pool에 올라오는 Page가 결국 이 파일에서 읽혀온다.

흐름을 읽는 방법:

1. **쿼리 진입** — 애플리케이션이 쿼리를 보내면 InnoDB Storage Engine이 받는다.
2. **Buffer Pool 확인** — Storage Engine은 먼저 메모리(Buffer Pool)를 뒤진다. 원하는 Page가 있으면 디스크를 건드리지 않는다.
3. **Cache Miss → 디스크 읽기** — Page가 없으면(Miss) `.ibd` 파일에서 Page를 읽어 Buffer Pool에 올린다.
4. **변경은 메모리에 먼저** — UPDATE/DELETE 등 변경이 생기면 Buffer Pool의 Page를 수정하고 Dirty Page로 표시한다. 디스크는 아직 건드리지 않는다.
5. **WAL 선기록** — 변경과 동시에 Redo Log에 "무엇을 바꿨다"는 로그를 순차 기록한다. 서버가 꺼져도 이 로그로 복구할 수 있다.
6. **Flush** — Checkpoint 시점에 Dirty Page를 모아 `.ibd`에 한 번에 쓴다. 여기서 랜덤 I/O가 발생하지만, 모아서 쓰기 때문에 횟수가 줄어든다.

핵심 트레이드오프: **쓰기 성능(메모리 버퍼링)** vs **내구성(WAL)** 을 동시에 잡는 구조다.

---

## 9. Buffering, Immutability, Ordering — 스토리지 구조의 3가지 축

DB 스토리지 엔진의 설계 방향을 결정하는 3가지 핵심 개념이 있다. B-Tree와 LSM-Tree의 차이도 결국 이 세 축에서의 선택 차이다.

### Buffering (버퍼링)

변경 사항을 즉시 디스크에 쓰지 않고, 메모리에 모아뒀다가 한 번에 쓰는 전략이다.

Buffer Pool의 Dirty Page 관리가 대표적인 예다. 개별 쓰기를 즉시 반영하는 대신, 여러 변경을 메모리에 축적해 I/O 횟수를 줄인다.

LSM-Tree는 이 개념을 더 극단까지 밀어붙인다. 모든 쓰기를 일단 메모리(Memtable)에 받고, 일정 크기가 되면 디스크에 Sequential Write로 한꺼번에 내려쓴다. 이 방식으로 Random I/O를 거의 완전히 Sequential I/O로 바꾼다.

### Immutability (불변성)

한번 쓴 데이터는 수정하지 않는다. 변경이 필요하면 기존 데이터를 덮어쓰는 대신 새 버전을 별도로 기록한다.

Deletion Marker(tombstone)가 이 패턴의 예다 — 삭제 시 기존 레코드를 지우는 대신 "삭제됨" 표시를 새로 추가한다.

LSM-Tree의 SSTable도 Immutable하다. 한번 디스크에 내려쓴 SSTable은 수정하지 않는다. 같은 키의 새 버전이 생기면 새 SSTable에 추가 기록하고, Compaction 시점에 정리한다.

Immutability의 이점:
- Sequential Write만 발생 → HDD/SSD 모두에서 빠름
- 동시성 제어가 단순해진다 (수정 중인 데이터가 없으니 Lock 경합 감소)
- 자연스럽게 이전 버전이 유지되어 MVCC 구현에 유리

### Ordering (정렬)

데이터를 삽입 순서가 아닌 특정 키 순서로 정렬해서 저장하는 전략이다.

B-Tree가 이 전략의 핵심이다. 키 순서를 유지하기 때문에 범위 조회(BETWEEN, ORDER BY)가 Sequential I/O로 처리된다.

LSM-Tree의 SSTable도 내부적으로 키 순서로 정렬된다. Merge 시점에 여러 SSTable을 합치면서 전체 정렬을 유지한다.

---

이 세 개념의 조합이 스토리지 엔진의 성격을 결정한다:

- **B-Tree**: Ordering 중심 — 항상 정렬된 상태 유지, In-place 수정
- **LSM-Tree**: Buffering + Immutability 중심 — Sequential Write 최대화, 나중에 Compaction으로 정리

어느 쪽이 유리한가는 workload에 따라 다르다. 읽기가 많고 범위 조회가 많으면 B-Tree, 쓰기가 폭발적으로 많으면 LSM-Tree. Phase 2~3에서 각각 상세히 다룬다.

---

## 실습

### 실습 1: InnoDB Page 크기 확인
```sql
SHOW VARIABLES LIKE 'innodb_page_size';
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```
> 📷 **[직접 실행 결과 캡쳐 첨부]**

### 실습 2: Buffer Pool 상태 확인
```sql
SHOW ENGINE INNODB STATUS\G
```
> 📷 **["BUFFER POOL AND MEMORY" 섹션 캡쳐 첨부 — Buffer pool hit rate 수치 포함]**

### 실습 3: .ibd 파일 크기 관찰
```bash
ls -lh /var/lib/mysql/{database_name}/
```
> 📷 **[.ibd 파일 목록 캡쳐 첨부 — 파일 크기가 16KB 배수인지 확인]**

### 실습 4: Buffer Pool 크기 변경 후 성능 비교
```sql
SET GLOBAL innodb_buffer_pool_size = 32 * 1024 * 1024;  -- 32MB로 줄이기
```
> 📷 **[Buffer Pool 축소 전/후 쿼리 실행 시간 비교 캡쳐 첨부]**

---

## 체크리스트

- [ ] `innodb_page_size`가 16384인 것을 직접 확인했다
- [ ] Buffer Pool hit rate를 `SHOW ENGINE INNODB STATUS`에서 찾았다
- [ ] "왜 Page 단위인가"를 한 문장으로 설명할 수 있다
- [ ] "Buffer Pool Miss가 왜 느린가"를 설명할 수 있다
- [ ] "Selectivity가 낮으면 왜 Full Scan이 더 빠른가"를 설명할 수 있다
- [ ] "Dirty Page를 즉시 디스크에 쓰지 않는 이유"를 설명할 수 있다
- [ ] "HDD에서 Random I/O가 느린 이유"를 seek time으로 설명할 수 있다
- [ ] "InnoDB Secondary Index 조회가 2단계인 이유"를 설명할 수 있다
- [ ] "DELETE가 즉시 파일에서 제거하지 않는 이유"를 설명할 수 있다
- [ ] Buffering / Immutability / Ordering 각각이 뭘 의미하는지 설명할 수 있다

---

## 다음 단계

Phase 1: [DBMS 아키텍처](db-dbms-architecture.md)
- 쿼리가 들어와서 결과가 나올 때까지 어떤 컴포넌트를 거치는가
- Storage Engine 내부는 어떻게 구성되는가

Phase 2: B-Tree 내부 구조
- Page들이 어떤 자료구조로 구성되는가
- 왜 MySQL은 B-Tree를 선택했는가
