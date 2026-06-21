# Data Files and Index Files

데이터베이스가 단순한 파일 시스템과 다른 핵심 이유 중 하나는 데이터를 구현 특화된 포맷으로 구성한다는 점이다. 이를 통해 저장 효율, 접근 효율, 갱신 효율을 동시에 달성한다.

## 왜 분리하는가

데이터베이스 시스템은 데이터 파일(data files)과 인덱스 파일(index files)을 분리한다.

- 데이터 파일: 실제 레코드를 저장한다.
- 인덱스 파일: 레코드 메타데이터를 저장하고, 데이터 파일 내 레코드 위치를 가리킨다.

파일은 페이지(page)로 나뉜다. 페이지 크기는 보통 하나 또는 여러 개의 디스크 블록에 해당한다.

## 데이터 파일의 종류

데이터 파일은 구현 방식에 따라 세 가지로 나뉜다.

**Index-Organized Tables (IOT)**
데이터 레코드를 인덱스 자체 안에 저장한다. 레코드가 키 순서대로 저장되므로, 범위 스캔을 인덱스를 순차적으로 읽는 것만으로 수행할 수 있다. 인덱스를 탐색해 키를 찾으면 별도 파일을 열지 않아도 데이터가 있다. InnoDB의 클러스터드 인덱스가 이 방식이다.

**Heap Files**
레코드를 특별한 순서 없이 저장한다. 대부분 쓰기 순서대로 쌓인다. 새 페이지를 추가할 때 별도 재조직이 필요 없다. 대신 레코드를 찾으려면 별도 인덱스 구조가 필수다.

**Hashed Files**
레코드를 버킷에 저장하고, 키의 해시값으로 버킷을 결정한다. 버킷 내부는 append 순서 또는 키 순 정렬로 저장할 수 있다.

## 인덱스 파일과 Primary/Secondary Index

인덱스는 키를 데이터 파일 내 위치에 매핑하는 자료구조다.

**Primary Index**
데이터 파일 자체에 대한 인덱스. 보통 기본 키 또는 기본 키 집합으로 구성된다. 검색 키 하나당 유일한 항목을 가진다.

**Secondary Index**
기본 인덱스 이외의 모든 인덱스. 같은 레코드를 여러 필드로 탐색할 수 있게 한다. 검색 키 하나에 여러 항목이 존재할 수 있다.

**Clustered vs Nonclustered**
레코드의 물리적 저장 순서가 인덱스의 키 순서와 일치하면 clustered(군집) 인덱스, 아니면 nonclustered(비군집) 인덱스다. IOT는 정의상 클러스터드다.

## Primary Index as Indirection

세컨더리 인덱스가 데이터 레코드를 직접 참조할 것인지, 기본 인덱스를 거쳐 참조할 것인지는 구현마다 다르다.

직접 참조는 읽기 경로에서 I/O가 적지만, 레코드가 이동하면 모든 세컨더리 인덱스의 포인터를 갱신해야 한다. 쓰기가 많은 워크로드에서 비용이 크다.

기본 인덱스를 거치는 방식은 포인터 갱신 비용을 줄인다. 레코드가 이동해도 기본 인덱스만 갱신하면 된다. 대신 읽기 경로에서 기본 인덱스를 추가로 탐색하는 오버헤드가 생긴다. MySQL InnoDB가 이 방식을 사용한다.

> 📷 Figure 1-5 (책 p.21) — Storing data records in an index file versus storing offsets to the data file
> 📷 Figure 1-6 (책 p.23) — Referencing data tuples directly (a) versus using a primary index as indirection (b)

## Deletion Markers (Tombstones)

대부분의 현대 스토리지 시스템은 페이지에서 데이터를 즉시 물리적으로 삭제하지 않는다. 대신 삭제 마커(tombstone)를 사용한다. 삭제 마커는 키와 타임스탬프를 담고 있어 "이 레코드는 삭제됐다"는 사실을 나타낸다.

실제 공간 회수는 가비지 컬렉션 과정에서 이루어진다. 페이지를 읽어 살아있는 레코드만 새 위치에 쓰고, 삭제된 레코드는 버린다.

참고: dbms-architecture.md, buffering-immutability-ordering.md
