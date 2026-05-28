# debezium

**"DB의 binlog를 읽어서 변경 이벤트를 Kafka로 흘려주는 CDC 도구"**

Debezium은 Kafka Connect의 Source Connector로 실행된다. MySQL, PostgreSQL, MongoDB 등의 변경 로그를 구독하고, 변경 이벤트를 Kafka 토픽으로 발행한다.

---

## 핵심 동작

Debezium MySQL/MariaDB Connector는 복제 Replica처럼 동작한다. DB가 실제 Replica로 인식하게끔 복제 프로토콜을 구현했기 때문에, DB에 별도 에이전트를 설치할 필요가 없다.

```
MySQL/MariaDB
  → binlog 스트림 (복제 프로토콜)
  → Debezium Connector 수신
  → Kafka 토픽으로 발행
```

발행하는 이벤트에는 변경 전후 상태(`before` / `after`)와 변경 종류(`op: c/u/d/r`)가 담긴다.

---

## Offset과 전달 보장

Debezium은 binlog 파일명과 position을 offset으로 관리하고, Kafka 내부 토픽에 저장한다. 재시작해도 마지막 offset부터 이어서 읽는다.

offset 커밋 전에 장애가 나면 같은 이벤트를 다시 발행할 수 있다. 기본 보장은 At-Least-Once다.

---

## 스냅샷

처음 시작할 때 binlog만으로는 기존 데이터를 알 수 없다. 그래서 Debezium은 기본적으로 초기 스냅샷을 찍는다. 테이블 전체를 SELECT해서 `op: r` 이벤트로 발행한 뒤, 그 시점 이후 binlog를 이어서 읽는다.

---

## 한 줄 요약

> Debezium = MySQL/MariaDB binlog를 Replica처럼 읽어서 Kafka로 발행하는 Kafka Connect Source Connector.

참고: cdc.md
참고: offset.md
참고: at-least-once.md
참고: debezium-event.md
참고: debezium-snapshot.md
참고: kafka-connect.md
