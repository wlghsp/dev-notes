# Source / Transform / Sink 개념

데이터 파이프라인 도구(Vector, Kafka Connect, Logstash 등)에서 공통으로 사용하는 용어입니다.

---

## 기본 구조

```
Source  →  Transform  →  Sink
(출발지)    (가공)        (목적지)
```

| 용어 | 역할 | 설명 |
|------|------|------|
| **Source** | 출발지 | 데이터를 읽어오는 곳 |
| **Transform** | 가공 | 데이터를 필터링·변환·집계 |
| **Sink** | 목적지 | 데이터를 내보내는 곳 |

---

## 도구별 적용 예시

### Vector

```
prom_rw (source)                     ← Prometheus remote_write 수신
  → filter_container (transform)     ← 필요한 메트릭만 필터
  → pod_normalize (transform)        ← 필드 정규화
  → pod_reduce (transform)           ← 집계
  → pod_envelope (transform)         ← 최종 포맷 구성
  → kafka_container (sink)           ← Kafka 토픽으로 전송
```

`vector.yaml`의 `sinks` 섹션이 여기에 해당합니다.  
토픽이 3개(`openshift-metric-container` / `node` / `vm`)로 분리된 이유는 메트릭 종류마다 스키마가 다르고, 컨슈머가 필요한 토픽만 구독할 수 있도록 하기 위해서입니다.

### Kafka Connect

Kafka Connect에서는 Source/Sink가 **Connector 단위**로 구분됩니다.

| Connector 종류 | 방향 | 예시 |
|---------------|------|------|
| **Source Connector** | 외부 → Kafka | Debezium (DB 변경사항을 Kafka로) |
| **Sink Connector** | Kafka → 외부 | JDBC Sink (Kafka 토픽을 DB에 저장) |

```
[MySQL] → Debezium Source Connector → [Kafka Topic] → JDBC Sink Connector → [MariaDB]
```

---

## 핵심 정리

- **Sink**는 특정 기술의 용어가 아니라 **데이터 흐름에서 끝점(목적지)** 을 가리키는 일반 개념
- Vector, Kafka Connect, Logstash 등 거의 모든 파이프라인 도구가 같은 의미로 사용
- Source가 많아도 Sink는 하나일 수 있고, 반대로 Sink를 여러 개 두어 같은 데이터를 여러 목적지로 보낼 수도 있음
