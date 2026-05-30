# kafka-connect

**"Kafka와 외부 시스템을 연결하는 데이터 통합 프레임워크"**

Kafka Connect는 Kafka와 DB, 파일, 검색엔진 같은 외부 시스템 사이의 데이터 이동을 표준화한 프레임워크다. Debezium처럼 직접 연동 코드를 짜지 않아도, Connector 설정만으로 데이터를 주고받을 수 있다.

---

## 핵심 개념

Kafka Connect는 세 가지 레이어로 구성된다.

- Worker — Connector와 Task를 실행하는 JVM 프로세스. 단독(standalone) 또는 분산(distributed) 모드로 실행
- Connector — 어떤 시스템과 어떻게 연결할지 정의하는 논리 단위. 설정 파일에 해당
- Task — Connector 설정을 바탕으로 실제 데이터를 읽거나 쓰는 실행 단위. 병렬 처리가 가능하면 여러 Task로 분산

```
Kafka Connect Worker
  └── Connector (설정)
        └── Task 1 (실제 실행)
        └── Task 2
```

---

## Source vs Sink

Connector는 방향에 따라 두 가지다.

- Source Connector — 외부 시스템 → Kafka. 데이터를 Kafka로 끌어온다. Debezium이 여기 해당
- Sink Connector — Kafka → 외부 시스템. Kafka 이벤트를 DB나 검색엔진에 쓴다

---

## 왜 Debezium이 Kafka Connect 위에서 동작하는가

CDC는 binlog를 지속적으로 읽는 장기 실행 작업이다. Worker 관리, offset 저장, 장애 복구, 스케일 아웃 같은 인프라 관심사를 Kafka Connect가 처리해준다. Debezium은 binlog를 읽는 로직에만 집중할 수 있다.

---

## 한 줄 요약

> Kafka Connect = Connector(설정) + Task(실행)를 Worker에서 돌리는 Kafka 연동 프레임워크.

참고: debezium.md
참고: kafka-connect-offset.md
