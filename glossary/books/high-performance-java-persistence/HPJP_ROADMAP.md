# High-Performance Java Persistence Glossary 진도표

참고: High-Performance Java Persistence — Vlad Mihalcea (2015-2016)
원본 파일: assets/database/High-Performance.Java.Persistence.pdf

이 PDF는 245페이지까지만 있는 초기 버전으로, Part I~III(12장 Inheritance)까지만 포함한다. Part IV 이후(Fetching 심화, 2차 캐시, 동시성 제어 심화 등)는 이 파일에 없다.

이 로드맵은 책을 챕터 순서대로 읽어나가며 새로 생기는 키워드를 챕터 단위로 관리하기 위한 것이다.

목차는 PDF 본문에서 직접 추출했다.

---

## Part I — Introduction

### Chapter 1 — Preface

데이터 접근 계층의 중요성, JDBC/JPA/jOOQ로 이어지는 책의 구성, 데이터 접근 스킬 스택(SQL → JDBC → ORM).

(완료 — 개론 장이라 별도 키워드 파일 없음)

### Chapter 2 — Performance and Scaling

응답 시간과 처리량의 정의, 데이터베이스 커넥션 한계, 스케일 업/아웃(Master-Slave 복제, Multi-Master 복제, 샤딩).

(완료)
- usl.md — Universal Scalability Law, contention/coherency 계수, Nmax 공식
- replication-topology.md — 동기/비동기 복제(hot/warm standby), Multi-Master 충돌 회피/해결
- sharding-topology.md — horizontal partitioning과의 차이, 샤드 간 조인 금지, 지리적 분산 샤딩, 샤딩이 최후의 수단인 이유

## Part II — JDBC and Database Essentials

### Chapter 3 — JDBC Connection Management

DriverManager와 DataSource, 커넥션 풀링이 빠른 이유, 대기 이론 기반 커넥션 풀 용량 산정, 실전 커넥션 풀 모니터링 지표.

(아직 진행 전)

### Chapter 4 — Batch Updates

Statement/PreparedStatement 배칭, 배치 크기 선택 기준, 벌크 연산, 자동 생성 키 조회와 시퀀스.

(아직 진행 전)

### Chapter 5 — Statement Caching

Statement 생명주기(Parser, Optimizer, Executor), 서버 사이드/클라이언트 사이드 캐싱, 바인드 민감 실행 계획.

(아직 진행 전)

### Chapter 6 — ResultSet Fetching

ResultSet의 scrollability/changeability/holdability, fetching size, ResultSet 크기 제한(limit, max rows), 컬럼 수 문제.

(아직 진행 전)

### Chapter 7 — Transactions

원자성/일관성/격리성/지속성(ACID), 동시성 제어(2단계 락킹, MVCC), 이상 현상(Dirty write/read, Non-repeatable read, Phantom read, Read/Write skew, Lost update), 격리 수준, 읽기 전용 트랜잭션, 트랜잭션 경계, 분산 트랜잭션(2PC), 선언적 트랜잭션, 애플리케이션 레벨 트랜잭션, 낙관적/비관적 락킹.

(아직 진행 전)

## Part III — JPA and Hibernate

### Chapter 8 — Why JPA and Hibernate matter

임피던스 불일치, JPA vs Hibernate, 스키마 소유권, 쓰기 기반/읽기 기반 최적화.

(아직 진행 전)

### Chapter 9 — Connection Management and Monitoring

JPA/Hibernate 커넥션 제공자(DriverManager, C3P0, Hikari, DataSource), 커넥션 릴리즈 모드, Hibernate 통계 모니터링, 문장 로깅(DataSource-proxy, P6Spy).

(아직 진행 전)

### Chapter 10 — Mapping Types and Identifiers

기본/문자열/날짜/숫자/바이너리/UUID/커스텀 타입 매핑, UUID/숫자 식별자 생성 전략(Identity, Sequence, Table generator), hi/lo 알고리즘과 pooled optimizer, 식별자 생성기 성능 비교.

(아직 진행 전)

### Chapter 11 — Relationships

관계 타입, @ManyToOne, @OneToMany(단방향/양방향/정렬), @ElementCollection, @OneToOne, @ManyToMany.

(아직 진행 전)

### Chapter 12 — Inheritance

Single table, Join table, Table-per-class, Mapped superclass 전략과 각각의 트레이드오프.

(아직 진행 전)
