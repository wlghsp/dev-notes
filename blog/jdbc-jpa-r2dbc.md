# JDBC, JPA, R2DBC — Java 데이터 접근의 세 계층

> "Spring Data JPA 쓰면 되는 거 아니야?" 맞다. 그런데 JPA가 내부적으로 뭘 하는지, R2DBC는 왜 따로 존재하는지, JDBC는 어디서 끝나는지 모르면 — 성능 문제가 생겼을 때 어디를 봐야 할지 모른다.

---

## 전체 계층 구조

```
애플리케이션 코드
       ↓
Spring Data JPA  (객체-테이블 매핑, JPQL, 변경 감지)
       ↓
JPA (Hibernate)  (Entity 생명주기 관리, SQL 생성)
       ↓
JDBC             (Java ↔ DB 통신 표준 인터페이스)
       ↓
JDBC Driver      (DB 벤더별 구현체: MySQL, PostgreSQL, Oracle...)
       ↓
TCP 소켓 → DB 서버
```

R2DBC는 JDBC와 같은 계층에 있지만, 블로킹이 아닌 논블로킹 통신을 한다.

```
Spring Data R2DBC
       ↓
R2DBC           (논블로킹 DB 통신 표준)
       ↓
R2DBC Driver
       ↓
비동기 소켓 → DB 서버
```

---

## JDBC — Java와 DB 사이의 표준 계약

### JDBC가 나온 이유

1990년대 초, Java 코드에서 DB에 접근하는 방법이 DB마다 달랐다. MySQL용 코드가 Oracle에서 안 돌아갔다. 라이브러리 교체 == 코드 전면 수정.

Sun Microsystems가 1997년에 **JDBC(Java Database Connectivity)** 를 표준으로 내놨다. DB 벤더는 이 표준에 맞춰 드라이버를 만들고, 개발자는 드라이버가 바뀌어도 같은 코드를 쓴다.

```java
// DB가 MySQL이든 Oracle이든 이 코드는 동일하다
Connection conn = DriverManager.getConnection(url, user, password);
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
stmt.setLong(1, userId);
ResultSet rs = stmt.executeQuery();

while (rs.next()) {
    String name = rs.getString("name");
}

rs.close();
stmt.close();
conn.close();
```

### JDBC의 본질

JDBC는 **인터페이스 명세**다. `java.sql.Connection`, `java.sql.PreparedStatement`, `java.sql.ResultSet` — 이것들은 전부 인터페이스다.

실제 구현은 각 DB 벤더의 드라이버가 한다.

```
java.sql.Connection (인터페이스)
    ↑
com.mysql.cj.jdbc.ConnectionImpl (MySQL 드라이버 구현체)
com.postgresql.jdbc.PgConnection (PostgreSQL 드라이버 구현체)
oracle.jdbc.driver.OracleConnection (Oracle 드라이버 구현체)
```

### JDBC의 문제점

JDBC 코드는 **블로킹**이다. `executeQuery()`를 호출하면 DB가 응답할 때까지 스레드가 멈춘다.

```java
ResultSet rs = stmt.executeQuery();
// 이 줄에서 스레드가 DB 응답을 기다리며 멈춤
// DB가 100ms 후 응답하면, 그 100ms 동안 스레드는 아무것도 못 함
```

그리고 코드가 길다. 연결 열기, 쿼리 실행, 결과 매핑, 연결 닫기 — 반복 코드가 너무 많다.

이걸 해결하려고 나온 게 JPA다.

---

## JPA — SQL 말고 객체로 생각하자

### JPA가 나온 이유

JDBC 시대의 개발자는 두 가지 세계를 동시에 살았다.

```
Java 코드      →    객체 지향: User 객체, Order 객체, 관계...
DB 테이블      →    관계형: users 테이블, orders 테이블, JOIN...
```

이 둘 사이의 변환 코드(보일러플레이트)를 매번 직접 짰다.

```java
// ResultSet → User 객체 변환을 손으로...
User user = new User();
user.setId(rs.getLong("id"));
user.setName(rs.getString("name"));
user.setEmail(rs.getString("email"));
// 컬럼이 20개면 20줄...
```

**ORM(Object-Relational Mapping)** 의 아이디어: 이 변환을 자동화하자.

JPA(Java Persistence API)는 Java 표준 ORM 인터페이스다. 구현체는 **Hibernate**가 대표적이다.

```
JPA           → 표준 인터페이스 (javax.persistence.*)
Hibernate     → JPA 구현체 (가장 많이 쓰임)
Spring Data JPA → Hibernate 위에 Repository 패턴을 얹은 것
```

### JPA가 하는 것

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;
}
```

이제 테이블 → 객체 변환은 JPA가 한다.

```java
// 저장
userRepository.save(new User("홍길동", "hong@example.com"));

// 조회
User user = userRepository.findById(1L).orElseThrow();

// 수정 — SQL UPDATE를 직접 안 써도 됨
user.setName("홍길순");
// 트랜잭션 종료 시 Hibernate가 변경을 감지하고 UPDATE 쿼리 자동 실행
```

### JPA가 내부에서 하는 것 — 이게 중요하다

JPA는 SQL을 없애는 게 아니다. **SQL을 대신 생성하는 것**이다. 내부는 여전히 JDBC다.

```
userRepository.findById(1L)
    ↓
Hibernate: SELECT u.* FROM users u WHERE u.id = ?   ← SQL 생성
    ↓
JDBC executeQuery()                                  ← 블로킹 JDBC 호출
    ↓
ResultSet → User 객체                                ← 자동 변환
```

Spring Data JPA를 써도, 내부는 Hibernate가 SQL 만들고, JDBC가 실행한다.

---

## JPA에서 잘못 이해하는 것들

### 오해 1 — 영속성 컨텍스트(Persistence Context)가 캐시다?

반은 맞고 반은 틀리다.

영속성 컨텍스트는 **1차 캐시** 기능이 있다. 같은 트랜잭션 안에서 같은 ID를 두 번 조회하면 DB를 두 번 안 친다.

```java
// 같은 트랜잭션 안
User user1 = userRepository.findById(1L); // SELECT 쿼리 실행
User user2 = userRepository.findById(1L); // 쿼리 안 날림, 캐시에서 반환
System.out.println(user1 == user2); // true — 같은 인스턴스
```

하지만 **트랜잭션이 끝나면 1차 캐시도 사라진다.** 요청 간 공유되지 않는다. 글로벌 캐시로 착각하면 안 된다.

### 오해 2 — 변경 감지(Dirty Checking)는 항상 효율적이다

Hibernate는 트랜잭션 커밋 시 엔티티의 현재 상태와 최초 조회 시의 스냅샷을 비교해서 변경된 필드를 UPDATE한다.

```java
@Transactional
public void updateName(Long id, String newName) {
    User user = userRepository.findById(id).orElseThrow();
    user.setName(newName); // SQL 안 씀
} // 트랜잭션 종료 → Dirty Checking → UPDATE users SET name=? WHERE id=?
```

문제는 **컬럼이 많고 엔티티 수가 많을 때** 비교 비용이 생각보다 크다. 수천 개의 엔티티를 한 트랜잭션에서 수정하면, 커밋 시점에 수천 번의 스냅샷 비교가 일어난다.

대량 업데이트는 JPQL 벌크 연산이나 네이티브 쿼리를 쓰는 게 낫다.

```java
// 느림 — 엔티티 수만큼 UPDATE
users.forEach(u -> u.setActive(false));

// 빠름 — DB에서 한 번에 처리
@Modifying
@Query("UPDATE User u SET u.active = false WHERE u.createdAt < :date")
void deactivateOldUsers(@Param("date") LocalDateTime date);
```

### 오해 3 — N+1 문제는 JPA를 잘못 쓰는 것이다

N+1은 JPA의 Lazy Loading에서 자연스럽게 발생한다. 잘못 쓴 게 아니라 기본 동작의 결과다.

```java
// users 테이블에서 유저 10명 조회 → SELECT 1번
List<User> users = userRepository.findAll();

// 각 유저의 orders를 접근할 때마다 SELECT → 10번
for (User user : users) {
    System.out.println(user.getOrders().size()); // 여기서 쿼리 날아감
}
// 총 SELECT 11번 (1 + 10)
```

해결법:

```java
// FETCH JOIN으로 한 번에 조회
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id IN :ids")
List<User> findAllWithOrders(@Param("ids") List<Long> ids);

// 또는 @EntityGraph
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

N+1은 ORM이 만들어낸 문제가 아니라, **"언제 쿼리가 날아가는지 모르는 상태"** 에서 생기는 문제다.

### 오해 4 — JPA를 쓰면 SQL을 몰라도 된다

JPA가 생성하는 SQL을 눈으로 확인하지 않으면 문제가 생긴다.

```yaml
# 개발 중에는 반드시 쿼리 로그를 켜라
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE  # 바인딩 파라미터까지
```

Hibernate가 생성한 쿼리가 기대와 다를 수 있다. 특히 JOIN 방향, 인덱스 활용 여부, 페이징 쿼리의 count 쿼리 등.

---

## R2DBC — 논블로킹 DB 통신

### R2DBC가 나온 이유

Spring WebFlux로 논블로킹 웹 서버를 만들었는데, DB 쿼리에서 JDBC를 쓰면 이벤트루프 스레드가 블로킹된다.

```
WebFlux 이벤트루프 스레드
    → HTTP 요청 수신 (논블로킹 ✓)
    → 비즈니스 로직 (논블로킹 ✓)
    → JDBC executeQuery() 호출 ← 여기서 스레드가 멈춤 ✗
```

블로킹 코드를 우회하려면 `Schedulers.boundedElastic()`에 오프로드해야 하는데, 이건 결국 블로킹 I/O를 다른 스레드풀로 숨기는 것이다. 진짜 논블로킹이 아니다.

**R2DBC(Reactive Relational Database Connectivity)** 는 DB 드라이버 수준부터 비동기로 만든 표준이다.

```java
// JDBC — 블로킹
ResultSet rs = stmt.executeQuery(); // DB 응답까지 대기

// R2DBC — 논블로킹
Flux<User> users = r2dbcTemplate.query("SELECT * FROM users", ...); // 즉시 Flux 반환
// DB 응답이 오면 리액티브 파이프라인에서 처리
```

### R2DBC와 Spring Data R2DBC

```
Spring Data R2DBC  →  Repository 패턴 (Spring Data JPA와 유사한 API)
      ↓
   R2DBC           →  논블로킹 DB 통신 표준
      ↓
R2DBC Driver       →  r2dbc-mysql, r2dbc-postgresql, r2dbc-mssql...
      ↓
비동기 소켓 → DB
```

```java
// Spring Data R2DBC Repository
public interface UserRepository extends ReactiveCrudRepository<User, Long> {
    Flux<User> findByName(String name);
    Mono<User> findByEmail(String email);
}

// 사용
@Service
public class UserService {
    public Flux<User> getActiveUsers() {
        return userRepository.findAll()
            .filter(User::isActive);
    }
}
```

반환 타입이 `List` 대신 `Flux`, `Optional` 대신 `Mono`다.

---

## R2DBC에서 잘못 이해하는 것들

### 오해 1 — R2DBC는 JPA를 대체한다

R2DBC는 JDBC를 대체하는 것이지, JPA를 대체하는 게 아니다.

```
JPA (Hibernate) → JDBC 위에서 동작
R2DBC           → JDBC와 같은 계층, 별도 표준

JPA + R2DBC 조합은 불가능하다
```

Hibernate는 내부적으로 JDBC를 쓰도록 설계됐다. R2DBC 위에서 동작하지 않는다. R2DBC를 선택하면 JPA의 영속성 컨텍스트, 변경 감지, 연관관계 자동 로딩 같은 기능을 포기하는 것이다.

Spring Data R2DBC는 훨씬 얇다. Repository 인터페이스는 제공하지만 **1차 캐시도, Dirty Checking도, Lazy Loading도 없다.**

### 오해 2 — WebFlux + R2DBC를 쓰면 항상 빠르다

R2DBC가 논블로킹이라도, **DB가 빠르지 않으면 빠른 게 아니다.**

논블로킹의 이점은 **스레드를 낭비하지 않는 것**이다. CPU는 대기 중에 다른 요청을 처리할 수 있다. 하지만 쿼리 자체의 응답 시간이 줄어드는 건 아니다.

```
JDBC:   스레드 1개 점유, 100ms 대기 후 응답
R2DBC:  스레드 미점유, 100ms 후 콜백, 그 동안 스레드는 다른 요청 처리

응답 지연(Latency): 둘 다 100ms
처리량(Throughput): R2DBC가 높음 (스레드를 낭비하지 않으므로)
```

쿼리 최적화 없이 WebFlux + R2DBC로 마이그레이션해도 느린 쿼리는 여전히 느리다.

### 오해 3 — 모든 DB가 R2DBC를 지원한다

2025년 기준으로 R2DBC 드라이버 지원 현황:

| DB | 지원 | 비고 |
|---|---|---|
| PostgreSQL | ✓ | r2dbc-postgresql, 안정적 |
| MySQL | ✓ | r2dbc-mysql |
| MariaDB | ✓ | r2dbc-mariadb |
| MS SQL Server | ✓ | r2dbc-mssql |
| Oracle | △ | r2dbc-oracle, 지원하나 성숙도 낮음 |
| MongoDB | - | Reactive Streams로 별도 지원 |

Oracle, 일부 레거시 DB는 드라이버 성숙도가 낮거나 없다.

---

## JDBC vs R2DBC — 실제 차이

### 코드 비교

```java
// JDBC (Spring JdbcTemplate)
public List<User> findActiveUsers() {
    return jdbcTemplate.query(
        "SELECT * FROM users WHERE active = true",
        (rs, rowNum) -> new User(rs.getLong("id"), rs.getString("name"))
    );
}

// R2DBC (Spring Data R2DBC)
public Flux<User> findActiveUsers() {
    return r2dbcTemplate.query(
        "SELECT * FROM users WHERE active = true",
        (row, meta) -> new User(row.get("id", Long.class), row.get("name", String.class))
    );
}
```

코드 구조는 비슷하다. 차이는 반환 타입이 `List` vs `Flux`이고, 내부 I/O가 블로킹 vs 논블로킹이다.

### 스레드 동작 차이

```
JDBC:
  스레드 A → executeQuery() → [100ms 대기] → 결과 → 반환
  이 100ms 동안 스레드 A는 아무것도 못 함

R2DBC:
  스레드 A → query() 호출 → Flux 즉시 반환 → 스레드 A 해방
  [100ms 후] → 이벤트루프 스레드가 결과 처리 → 다음 operator 실행
```

### 언제 어떤 걸 선택하는가

```
JDBC / Spring Data JPA가 맞는 경우:
  - 일반 CRUD 서비스
  - 복잡한 도메인 모델 (연관관계, 상속 구조)
  - 팀이 JPA에 익숙할 때
  - 동시 접속이 많지 않을 때

R2DBC / Spring Data R2DBC가 맞는 경우:
  - Spring WebFlux 기반으로 전체 스택 논블로킹
  - 동시 접속이 매우 많은 서비스 (API Gateway, 실시간 서비스)
  - DB 쿼리 + 외부 API 호출이 혼재해 비동기 합성이 필요할 때
  - JPA 없이도 충분한 단순한 데이터 모델
```

---

## 트랜잭션 — 블로킹과 논블로킹의 차이

JDBC의 트랜잭션은 **스레드**에 묶인다. 같은 스레드에서 커밋이나 롤백이 일어난다.

```java
// Spring @Transactional (JDBC 기반)
@Transactional
public void transfer(Long from, Long to, BigDecimal amount) {
    // 이 메서드 전체가 하나의 스레드에서 실행됨
    // 트랜잭션 컨텍스트가 ThreadLocal에 저장됨
    account.debit(from, amount);
    account.credit(to, amount);
}
```

R2DBC의 트랜잭션은 **리액티브 컨텍스트(Reactor Context)** 에 묶인다. 스레드가 바뀌어도 트랜잭션이 유지된다.

```java
// Spring @Transactional (R2DBC 기반)
@Transactional
public Mono<Void> transfer(Long from, Long to, BigDecimal amount) {
    return accountRepository.debit(from, amount)
        .then(accountRepository.credit(to, amount));
    // 실행 도중 스레드가 바뀔 수 있지만 트랜잭션은 유지됨
    // Reactor Context에 트랜잭션 정보가 전파됨
}
```

이 차이 때문에 R2DBC 환경에서 ThreadLocal을 직접 쓰는 코드(MDC 로깅, 일부 보안 라이브러리)는 트랜잭션 전파가 제대로 안 될 수 있다.

---

## 정리

```
JDBC
  └─ Java ↔ DB 통신의 표준 인터페이스
  └─ 블로킹. 스레드가 DB 응답을 기다리며 대기
  └─ SQL을 직접 다룬다

JPA (Hibernate)
  └─ JDBC 위에 얹힌 ORM
  └─ 객체-테이블 자동 매핑, SQL 자동 생성
  └─ 영속성 컨텍스트, Dirty Checking, Lazy Loading
  └─ N+1, 대량 업데이트 주의. 생성 SQL을 반드시 확인할 것

R2DBC
  └─ JDBC를 대체하는 논블로킹 DB 통신 표준
  └─ Flux/Mono로 반환. 스레드를 점유하지 않음
  └─ JPA 불가. 영속성 컨텍스트 없음
  └─ WebFlux 전체 스택과 함께 쓸 때 의미가 있다
```

> JPA를 쓰면서 WebFlux를 붙이는 건 반쪽짜리다. DB 레이어에서 블로킹이 일어나니까. 논블로킹을 제대로 하려면 R2DBC까지 가야 한다. 그 대신 JPA의 편의 기능은 포기해야 한다.
>
> 어떤 걸 선택할지는 서비스의 동시성 요구와 도메인 복잡도를 저울질해서 결정한다. 둘 다 좋다고 섞으면 둘 다 어렵다.
