# DBMS Architecture

데이터베이스 관리 시스템은 공통된 청사진이 없다. 데이터베이스마다 약간씩 다르게 구현되고, 컴포넌트 경계가 명확하지 않은 경우도 많다. 하지만 대부분의 DBMS는 공통된 계층 구조를 공유한다.

> 📷 Figure 1-1 (책 p.8) — Architecture of a database management system

## 전체 구조

클라이언트 요청이 들어오면 다음 순서로 처리된다.

1. **Transport** — 클라이언트 요청을 받아들이는 진입점. 클러스터 내 다른 노드와의 통신도 담당한다.
2. **Query Processor** — 쿼리를 파싱하고 검증한다. 파싱 후 접근 제어를 수행하고, Query Optimizer로 넘긴다.
3. **Query Optimizer** — 불필요하거나 불가능한 쿼리 일부를 제거하고, 가장 효율적인 실행 계획을 선택한다. 인덱스 카디널리티, 데이터 위치 등 내부 통계를 활용한다.
4. **Execution Engine** — 실행 계획을 받아 로컬 실행과 리모트 실행을 조율한다. 리모트 실행에는 다른 노드에서 읽고 쓰는 것, 복제 등이 포함된다.
5. **Storage Engine** — 로컬 쿼리를 실제로 처리하는 핵심 컴포넌트.

## Storage Engine의 내부 컴포넌트

스토리지 엔진 안에는 별도 책임을 가진 4개의 서브컴포넌트가 있다.

**Transaction Manager**
트랜잭션 스케줄링을 담당한다. 트랜잭션이 데이터베이스를 논리적으로 일관되지 않은 상태로 남기지 않도록 보장한다.

**Lock Manager**
실행 중인 트랜잭션이 데이터베이스 객체에 락을 거는 작업을 관리한다. 동시 연산이 물리적 데이터 무결성을 침해하지 않도록 보장한다.

Transaction Manager와 Lock Manager는 함께 동시성 제어(Concurrency Control)를 담당한다. 논리적 무결성과 물리적 무결성을 동시에 보장하면서 동시 연산이 최대한 효율적으로 실행되도록 한다.

**Access Methods (Storage Structures)**
디스크에서 데이터를 접근하고 구성하는 방법을 관리한다. 힙 파일이나 B-Tree, LSM Tree 같은 스토리지 구조가 여기에 해당한다.

**Buffer Manager**
데이터 페이지를 메모리에 캐시한다. 디스크 I/O를 줄이기 위해 자주 접근하는 페이지를 메모리에 유지한다.

**Recovery Manager**
연산 로그를 유지하고, 장애 발생 시 시스템 상태를 복원한다. WAL(Write-Ahead Logging)이 이 계층에서 동작한다.

## 핵심 통찰

컴포넌트 경계는 이론상 명확하지만, 실제 코드에서는 성능 최적화나 엣지 케이스 처리 때문에 경계가 흐려지는 경우가 많다. 이 구조를 이해하는 목적은 "어떤 책임이 어디에 있는가"를 파악하는 것이지, 모든 DB가 이 구조를 그대로 따른다고 가정하는 것이 아니다.

참고: memory-vs-disk-dbms.md, data-files-and-index-files.md
