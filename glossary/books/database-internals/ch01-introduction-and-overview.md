# Chapter 1 — Introduction and Overview

참고: Database Internals — Alex Petrov (O'Reilly, 2019)

이 챕터에서 생성된 키워드 파일:
- dbms-architecture.md
- memory-vs-disk-dbms.md
- column-vs-row-oriented.md
- data-files-and-index-files.md
- buffering-immutability-ordering.md

---

## 챕터의 목적

이 챕터는 이후 모든 챕터를 읽기 위한 지도다. 데이터베이스 시스템을 어떤 기준으로 분류하는지, 스토리지 엔진이 무엇인지, 그리고 스토리지 엔진 설계의 핵심 축이 무엇인지를 정의한다.

---

## DBMS Architecture

모든 DBMS는 공통된 레이어 구조를 가진다.

클라이언트 요청은 Transport 레이어로 들어온다. Query Processor가 파싱하고 검증한다. Query Optimizer가 가장 효율적인 실행 계획을 선택한다. Execution Engine이 계획을 실행한다. 로컬 쿼리는 Storage Engine이 처리한다.

Storage Engine 내부에는 Transaction Manager, Lock Manager, Access Methods, Buffer Manager, Recovery Manager가 있다. Transaction Manager와 Lock Manager가 함께 동시성 제어를 담당한다.

> 📷 Figure 1-1 (책 p.8) — Architecture of a database management system

---

## Memory vs Disk-Based DBMS

인메모리 DBMS는 데이터를 주로 RAM에 저장하고, 디스크는 복구와 로깅에만 쓴다. 디스크 기반 DBMS는 데이터를 주로 디스크에 저장하고, 메모리는 캐시로 쓴다.

인메모리 DB의 약점은 RAM의 휘발성이다. 전원이 꺼지면 모든 데이터가 사라진다. 이를 해결하기 위해 WAL(Write-Ahead Log)로 모든 연산을 순차 로그에 기록하고, 주기적으로 checkpointing으로 스냅샷을 디스크에 저장한다.

인메모리 DB는 디스크 기반 DB에 큰 캐시를 얹은 것이 아니다. 사용하는 자료구조와 최적화 방법이 근본적으로 다르다.

---

## Column vs Row-Oriented DBMS

로우 지향은 같은 로우의 값을 연속으로 저장한다. 전체 레코드를 읽고 쓰는 OLTP에 적합하다.

컬럼 지향은 같은 컬럼의 값을 연속으로 저장한다. 특정 컬럼의 집계를 계산하는 OLAP에 적합하다. 같은 타입 데이터가 연속으로 쌓이므로 압축 효율이 높고, CPU SIMD 명령어 활용이 가능하다.

선택 기준은 접근 패턴이다. 대부분의 컬럼을 쓰는 포인트 쿼리라면 로우 지향, 일부 컬럼의 집계라면 컬럼 지향이 낫다.

> 📷 Figure 1-2 (책 p.17) — Data layout in column- and row-oriented stores

---

## Data Files and Index Files

데이터베이스는 데이터를 구현 특화 포맷으로 구성한다. 크게 데이터 파일과 인덱스 파일로 분리한다.

데이터 파일의 세 가지 종류:
1. Index-Organized Tables (IOT) — 인덱스 안에 데이터를 직접 저장. 범위 스캔이 빠르다
2. Heap Files — 순서 없이 저장. 별도 인덱스가 필수
3. Hashed Files — 키 해시값으로 버킷을 결정

인덱스는 Primary Index(기본 키 기반, 레코드당 유일한 항목)와 Secondary Index(그 외, 여러 항목 가능)로 나뉜다. 레코드의 물리적 순서가 인덱스 키 순서와 일치하면 Clustered, 그렇지 않으면 Nonclustered다.

세컨더리 인덱스가 데이터를 직접 참조하면 읽기가 빠르지만 레코드 이동 시 모든 포인터를 갱신해야 한다. 기본 인덱스를 거쳐 간접 참조하면 갱신 비용이 줄지만 읽기 경로가 늘어난다. InnoDB는 후자를 선택했다.

대부분의 현대 스토리지 시스템은 삭제 시 즉시 공간을 회수하지 않고 tombstone 마커를 남긴다. 가비지 컬렉션 때 실제로 정리한다.

> 📷 Figure 1-5 (책 p.21) — Storing data records in an index file versus storing offsets to the data file
> 📷 Figure 1-6 (책 p.23) — Referencing data tuples directly (a) versus using a primary index as indirection (b)

---

## Buffering, Immutability, and Ordering

스토리지 엔진 설계의 모든 차이는 이 세 축으로 설명된다.

**Buffering** — 디스크에 쓰기 전에 메모리에 데이터를 모을 것인가. LSM Tree는 인메모리 버퍼를 적극 활용한다. B-Tree는 기본적으로 버퍼링 없이 즉시 쓴다.

**Immutability** — 기존 파일 내용을 수정하는가(mutable), 아니면 새 데이터를 끝에만 추가하는가(immutable). B-Tree는 가변, LSM Tree의 SSTable은 불변이다.

**Ordering** — 데이터가 키 순서대로 저장되는가. 순서를 유지하면 범위 스캔이 효율적이다. 순서를 버리면(삽입 순서 저장) 쓰기가 빠르다.

이 세 축의 조합이 B-Tree, LSM Tree, Lazy B-Tree, Copy-on-Write Tree 등 다양한 스토리지 구조를 만들어낸다.

---

## 핵심 통찰

이 챕터의 핵심은 "최적의 스토리지 엔진은 없다"는 것이다. 각 구조는 특정 트레이드오프를 선택한 결과다. B-Tree가 왜 디스크 기반의 표준이 됐는지, LSM Tree가 왜 쓰기 집약적 환경에서 주목받는지를 이해하는 틀이 이 챕터에서 만들어진다.
