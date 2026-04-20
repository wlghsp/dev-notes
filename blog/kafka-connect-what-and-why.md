# Kafka Connect란 무엇인가 — 데이터 파이프라인의 자동화

> 데이터베이스의 변경사항을 Kafka로 보내고, Kafka의 메시지를 다른 곳에 저장하고 싶다. 근데 매번 코드를 짜야 하나? 아니다. **Kafka Connect가 자동으로 해준다.**

---

## 1. Kafka Connect가 무엇인가

### 정의

**Kafka Connect는 Java로 만든 독립 서버/애플리케이션**입니다.

역할: 데이터 소스(MySQL, PostgreSQL 등)와 Kafka를 **자동으로 연결**해주는 중개자입니다.

### 프로그램의 종류

- 📦 **Java 애플리케이션** (독립 실행형 JVM 프로그램)
- 🔌 **플러그인 기반** (Connector 플러그인을 로드해서 작동)
- 🔧 **설정 기반** (코드가 아닌 JSON으로 제어)
- 🌐 **REST API** (HTTP로 관리/제어 가능)

### 구조

```
┌─────────────────────────────────────────┐
│     Kafka Connect (서버)                 │
├─────────────────────────────────────────┤
│                                         │
│  [Connector 플러그인들]                 │
│  ├─ Debezium MySQL Connector            │
│  ├─ JDBC Sink Connector                 │
│  ├─ S3 Connector                        │
│  └─ Elasticsearch Connector             │
│                                         │
│  [Worker (실행 엔진)]                   │
│  ├─ Task 관리 & 실행                    │
│  ├─ 병렬 처리                           │
│  ├─ 장애 복구                           │
│  └─ Offset 추적                         │
│                                         │
│  [REST API (관리)]                      │
│  ├─ Connector 등록/삭제/제어             │
│  └─ 상태 모니터링                       │
│                                         │
└─────────────────────────────────────────┘
```

### Before: 수동으로 코드를 짜는 방식

```
개발자가 직접 작성해야 할 일:

MySQL 데이터
    ↓
(Java/Python 코드로 직접 구현)
  ├─ MySQL 연결
  ├─ 데이터 주기적으로 읽기
  ├─ 에러 처리
  ├─ 재시도 로직
  ├─ 로깅
  └─ Kafka에 전송
    ↓
Kafka
    ↓
(또 다른 코드 작성)
  ├─ Kafka 구독
  ├─ Target 연결
  ├─ 데이터 변환
  ├─ 에러 처리
  ├─ 재시도 로직
  └─ Target에 저장
    ↓
Target (MySQL, Elasticsearch 등)

⏱️ 시간 오래 걸림 (개발, 테스트, 배포, 운영 모두 수동)
```

### After: Kafka Connect 방식

```
JSON 설정파일만 작성

{
  "name": "mysql-source",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql",
    ...
  }
}
    ↓
REST API로 Kafka Connect에 등록
    ↓
Kafka Connect가 자동으로:
  ├─ Connector 플러그인 로드
  ├─ MySQL 연결
  ├─ 데이터 자동 읽기
  ├─ 에러 자동 처리
  ├─ 재시도 자동 실행
  ├─ Offset 자동 추적
  └─ Kafka로 자동 전송
    ↓
Kafka
    ↓
또 다른 Consumer (Logstash 등)
  ├─ 자동으로 메시지 읽기
  ├─ Target에 자동 저장
  └─ 모든 처리 자동
    ↓
Target (MySQL, Elasticsearch 등)

⚡ 빠르고 간단함! (설정만 하면 끝)
```

### 핵심 특징

| 항목 | 설명 |
|------|------|
| **프로그램 형태** | Java 애플리케이션 (독립 서버) |
| **동작 방식** | Connector 플러그인 로드 & 실행 |
| **설정 방법** | JSON 파일 |
| **관리 방법** | REST API (HTTP) |
| **에러 처리** | 자동으로 모두 담당 |
| **코드 작성** | 0줄! (설정만 하면 됨) |

---

## 2. 핵심: Source → Kafka → Consumer

```
Source (MySQL)
    ↓ (Binlog 모니터링)
Source Connector (Debezium)
    ↓
Kafka Topic
    ↓
Consumer (Logstash, Sink Connector 등)
    ↓
Target (MySQL, Elasticsearch 등)
```

**Source Connector:** MySQL의 변경사항을 Kafka로 보냄  
**Consumer:** Kafka의 메시지를 읽어서 Target에 저장

---

## 3. 실제 동작 흐름

**1) Source MySQL 변경**
```
INSERT INTO users VALUES (1, 'Alice', 'alice@example.com')
```

**2) Kafka Topic에 자동 발행**
```json
{
  "after": {"id": 1, "name": "Alice", "email": "alice@example.com"},
  "op": "c"
}
```

**3) Target에 자동 반영 (Logstash 또는 Sink Connector)**
```
users 테이블에 (1, 'Alice', 'alice@example.com') 추가됨
```

**같은 방식으로 UPDATE/DELETE도 자동 동기화**

---

## 4. 장점 정리

- ✅ **설정만 하면 끝** (코드 불필요)
- ✅ **자동 에러 처리** (재시도 자동)
- ✅ **정확한 복구** (Offset으로 손실/중복 방지)
- ✅ **여러 곳으로 동시 전달** 가능
- ✅ **실시간 모니터링** (REST API)

---

## 5. 정리

| 용어 | 의미 |
|------|------|
| **Kafka Connect** | 설정 기반 데이터 동기화 도구 |
| **Source Connector** | 데이터 소스 → Kafka |
| **Offset** | 처리 위치 추적 (복구의 핵심) |
| **CDC** | 데이터 변경사항을 실시간으로 감지 |

> Kafka Connect로 MySQL 동기화를 자동화한다. 설정만 하면 계속 알아서 동작한다.

---

**더 자세한 내용은** [2편 - Kafka Connect 심화 개념](kafka-connect-advanced-concepts.md) **참고**
