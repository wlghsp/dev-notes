# PostgreSQL: Database vs Schema

**한 줄 정의**: Database는 완전히 독립된 물리적 공간이고, Schema는 그 안에서 객체를 논리적으로 나누는 구획이다.

---

## 계층 구조

```
PostgreSQL 서버
└── database (예: myapp)
    ├── schema: public
    │   ├── table: users
    │   └── table: orders
    ├── schema: analytics
    │   └── table: events
    └── schema: billing
        └── table: invoices
```

database가 집이라면, schema는 방이다. 방끼리는 문이 열려 있지만, 집 밖은 담이 막혀 있다.

---

## Database — 독립된 공간

database는 연결 단위다. 접속할 때 어느 database에 붙을지 지정해야 한다.

```
psql -d myapp
psql -d analytics_db
```

다른 database의 테이블은 직접 조회할 수 없다.

```sql
-- myapp에 접속한 상태에서
SELECT * FROM other_db.public.users;  -- 에러. 불가능
```

database를 넘나들려면 dblink나 foreign data wrapper 같은 별도 수단이 필요하다. 이 경계는 의도적인 것이다 — 서비스 간 결합을 막는 격리 수단으로 쓴다.

---

## Schema — 같은 DB 안의 논리적 구획

schema는 같은 database 안에서 자유롭게 넘나들 수 있다.

```sql
-- 같은 DB 안의 다른 schema 조회
SELECT * FROM analytics.events;
SELECT * FROM billing.invoices;
SELECT * FROM public.users;
```

schema를 명시하지 않으면 `search_path`에 따라 자동 탐색된다. 기본값은 `public`.

```sql
-- 현재 search_path 확인
SHOW search_path;

-- search_path 변경 (세션 단위)
SET search_path TO analytics, public;
```

`search_path`가 `analytics, public`이면 `SELECT * FROM events`라고 써도 먼저 `analytics.events`를 찾는다.

---

## 왜 schema를 나누는가

하나의 database에 테이블이 수십 개가 넘어가면 구분이 필요해진다.

실무에서 자주 쓰는 패턴은 두 가지다.

첫 번째는 도메인별 분리다. `auth.users`, `billing.invoices`, `analytics.events` 처럼 서비스 도메인 경계를 schema로 표현한다. 같은 DB에 있지만 논리적으로 분리되어 있다는 걸 명시한다.

두 번째는 권한 분리다. schema 단위로 접근 권한을 줄 수 있다. 분석팀에는 `analytics` schema만 읽기 권한을 주고, `billing` schema는 차단하는 식이다.

```sql
GRANT USAGE ON SCHEMA analytics TO analyst_role;
GRANT SELECT ON ALL TABLES IN SCHEMA analytics TO analyst_role;
```

---

## MySQL / MariaDB와 다른 점

MySQL과 MariaDB는 database와 schema가 동의어다. `CREATE DATABASE foo`와 `CREATE SCHEMA foo`가 완전히 같은 동작이고, 실무에서도 database를 "스키마"라고 부르는 경우가 많다.

그래서 PostgreSQL을 먼저 쓰다가 MariaDB로 넘어오면 혼선이 생긴다. PostgreSQL에서 schema는 database 안의 하위 구획인데, MariaDB에서 누군가 "스키마"라고 말하면 그건 database 자체를 가리키는 것이다. 같은 단어가 다른 레이어를 뜻한다.

체감 기준으로 대응 관계를 정리하면 이렇다.

```
PostgreSQL schema  ≈  MariaDB database
PostgreSQL database  ≈  (MariaDB에는 이 레이어가 없다)
```

PostgreSQL에서 "같은 DB에 접속한 채로 analytics.events, billing.invoices를 넘나들던" 그 동작이, MariaDB에서는 "하나의 연결로 analytics_db.events, billing_db.invoices를 넘나드는" 동작에 해당한다. 넘나드는 단위가 schema에서 database로 한 레이어 올라간 것이다.

PostgreSQL은 다르다. database > schema > table 의 3단 계층이 존재한다. MySQL / MariaDB만 쓰다 PostgreSQL로 넘어오면 이 계층을 무시하고 `public` schema에 모든 테이블을 몰아넣는 경우가 많은데, 그렇게 해도 동작은 하지만 schema를 사용하는 이유를 버리는 셈이다.

---

## 관련 개념
- 참고: isolation-level.md (database 간 격리와 transaction 격리는 다른 레이어)
