# Prometheus란 무엇인가 — 개념부터 실제 파이프라인까지

> 모니터링 도구를 고를 때 Prometheus라는 이름을 자주 듣는다. 근데 정작 "Prometheus가 뭐야?"라고 물으면 막히는 경우가 많다. 개념부터 실제로 어떻게 썼는지까지 정리했다.

---

## 1. Prometheus가 뭔가

**시스템의 상태를 숫자로 수집하고 저장하는 모니터링 도구**다.

여기서 숫자가 곧 **메트릭(Metric)** 이다.

```
CPU 사용률: 72%
JVM 힙 메모리: 1.2GB
HTTP 요청 수: 초당 340건
DB 커넥션 풀 사용 중: 18개
```

이런 숫자들을 시간 흐름에 따라 계속 수집해서 저장한다. 나중에 "언제 CPU가 튀었는가", "요청이 갑자기 늘어난 시점이 언제인가"를 추적할 수 있다.

### Prometheus는 왜 Go로 만들어졌나

Prometheus는 **Go(Golang)로 만들어졌다.** 2012년 SoundCloud에서 개발했고, 이후 오픈소스로 공개되어 2016년에 CNCF(Cloud Native Computing Foundation)에 합류했다.

Go를 선택한 이유는 꽤 합리적이다:
- **뛰어난 동시성 처리** — 고루틴(Goroutine)으로 수많은 메트릭 수집을 병렬로 처리한다
- **메모리 효율** — 메트릭 수집/저장 같은 I/O 집약적인 작업에 최적화되어 있다

덕분에 리소스를 적게 써서, 어떤 환경에서도 부담 없이 설치할 수 있다. 이게 현대적인 인프라 모니터링의 표준이 된 이유 중 하나다.

---

## 2. Pull 방식 — Prometheus의 핵심 특징

모니터링 도구에는 두 가지 방식이 있다.

**Push 방식** — 애플리케이션이 직접 모니터링 서버로 데이터를 밀어넣는다.

```
앱 → (데이터 전송) → 모니터링 서버
```

**Pull 방식** — 모니터링 서버가 직접 앱에 와서 데이터를 가져간다.

```
Prometheus → (주기적으로 스크래핑) → 앱의 /metrics 엔드포인트
```

Prometheus는 **Pull 방식**이다. 앱은 `/metrics` 경로에 현재 상태를 숫자로 노출해두기만 하면 된다. Prometheus가 설정한 주기(기본 15초)마다 와서 가져간다.

### Pull 방식의 장점

```
앱이 죽으면? → Prometheus가 스크래핑 실패 → "이 앱 응답 없음" 감지
앱이 살아있으면? → 정상 스크래핑 → 메트릭 수집 계속

Push 방식이었다면 앱이 죽으면 그냥 조용히 데이터가 안 들어온다.
Pull은 Prometheus가 직접 확인하므로 장애 감지가 더 명확하다.
```

---

## 3. PromQL — Prometheus 쿼리 언어

저장된 메트릭을 조회하는 쿼리 언어다.

```promql
# 지난 5분간 초당 HTTP 요청 수
rate(http_requests_total[5m])

# JVM 힙 사용률 (%)
jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"} * 100
```

### PromQL은 어디서 쓰는가

PromQL 쿼리 자체는 세 가지 곳에서 실제로 사용된다.

**1) Grafana 대시보드 (시각화)**

```
Grafana 대시보드
  └─ 패널(Panel) 생성
      └─ Data Source: Prometheus 선택
          └─ PromQL 입력: rate(http_requests_total[5m])
              └─ 그래프로 자동 시각화
```

운영팀이나 개발자가 보는 모니터링 화면은 거의 항상 Grafana다. Grafana에 PromQL 쿼리를 저장해두고 실시간 그래프를 본다.

> **Grafana vs Kibana 구분**
>
> | 구분 | Grafana | Kibana |
> |---|---|---|
> | **데이터 소스** | Prometheus, InfluxDB, MySQL 등 다양 | Elasticsearch만 가능 |
> | **쿼리 언어** | PromQL, SQL, Flux 등 | Kibana Query Language (KQL), PromQL도 가능 |
> | **주요 용도** | **실시간 모니터링** (CPU, 메모리, 요청 수 등) | **로그/메트릭 장기 분석** (로그 검색, 트렌드 분석) |
> | **우리 프로젝트에서** | Prometheus 메트릭을 실시간으로 봄 | Elasticsearch에 저장된 로그/메트릭을 분석함 |
>
> 즉, **Prometheus → Grafana (실시간)**, **Logstash → Elasticsearch → Kibana (장기 저장 & 분석)**이 흐름이다.

**2) Prometheus 알림 규칙**

```yaml
# /etc/prometheus/alerts.yml (또는 /prometheus/rules/alerts.yml)
groups:
  - name: application_alerts
    rules:
      - alert: HighCPUUsage
        expr: node_cpu_usage_percent > 80  # PromQL
        for: 5m
        annotations:
          summary: "CPU 사용률이 80% 이상입니다"
```

임계값 초과 시 자동으로 Slack, 이메일, PagerDuty 등으로 알림을 보낸다.

> **alerts.yml과 Alertmanager**
>
> **alerts.yml**은 Prometheus 설치 후 기본으로 제공되지 않는다. 운영자가 필요에 따라 직접 만들어야 한다.
> **Alertmanager**도 별도로 설치해야 한다 (Prometheus와는 독립적인 서버).
>
> ```yaml
> # /etc/prometheus/prometheus.yml (Prometheus 메인 설정)
> global:
>   scrape_interval: 15s
>
> rule_files:
>   - "alerts.yml"  # ← alerts.yml을 여기서 참조
>
> alerting:
>   alertmanagers:
>     - static_configs:
>         - targets: ['localhost:9093']  # Alertmanager 주소
>
> scrape_configs:
>   - job_name: 'node'
>     static_configs:
>       - targets: ['localhost:9100']
> ```
>
> **역할 분담:**
>
> | 역할 | 설명 |
> |---|---|
> | **Prometheus** | alerts.yml의 규칙을 주기적으로 평가. 조건 만족 시 알림 발생 |
> | **Alertmanager** | Prometheus에서 온 알림을 받아서 **중복 제거, 그룹화, 라우팅** 한 후 최종 전송 |
>
> 즉, **Prometheus = 감시자** (조건 체크), **Alertmanager = 알림 배달원** (어디로 보낼지 결정)이다.
>
> ```
> Prometheus (alerts.yml 평가)
>   └─ "CPU 80% 초과" 알림 발생
>       └─ Alertmanager로 전송
>           ├─ 중복 제거 (같은 알림이 여러 번 오면 하나로)
>           ├─ 그룹화 (관련 알림끼리 묶기)
>           └─ 라우팅 (Slack/Email/PagerDuty로 보낼지 결정)
> ```
>
> Alertmanager 설정(alertmanager.yml)에서 알림이 어디로 가는지 정의한다.

**3) Prometheus 웹 콘솔 (즉시 조회)**

Prometheus UI(`localhost:9090`)의 쿼리 탭에서 직접 입력해 즉시 결과를 본다. 대시보드가 없을 때나 임시로 메트릭을 확인할 때 쓴다.

---

## 4. Exporter — 메트릭을 노출하는 방법

앱은 코드에 Prometheus 라이브러리를 넣어서 `/metrics`를 직접 만들 수 있다. 하지만 베어메탈이나 VM처럼 코드를 수정할 수 없는 대상은 **Exporter**를 설치해서 메트릭을 노출한다. Exporter가 내부 상태를 읽어 Prometheus 포맷으로 변환해주면, Prometheus는 평소처럼 스크래핑하면 된다.

```
베어메탈/VM
    └── Node Exporter 설치 (데몬으로 실행)
            ↓ /proc, /sys 에서 OS 메트릭 읽기
        /metrics 노출
            ↓
        Prometheus 스크래핑
```

### 주요 Exporter

```
Node Exporter    — 서버(베어메탈/VM) CPU, 메모리, 디스크, 네트워크
JMX Exporter     — JVM 메트릭 (Java 앱)
MySQL Exporter   — MySQL 쿼리 수, 커넥션 수 등
Redis Exporter   — Redis 메모리, 명령 수 등
```

> **Node Exporter의 Node는 Node.js와 무관하다.** 여기서 Node는 "네트워크 상의 서버 한 대(노드)"를 뜻한다. Kubernetes에서 Node가 서버 한 대를 의미하는 것과 같은 맥락이다.

실무에서 베어메탈/VM을 모니터링한다면 **Node Exporter를 각 서버에 설치하는 것이 첫 번째 단계**다.

### Spring Boot Exporter

Spring Boot는 `spring-boot-actuator` + `micrometer-registry-prometheus` 의존성만 추가하면 `/actuator/prometheus` 엔드포인트가 자동으로 생긴다.

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

---

## 5. 실제로 이렇게 썼다 — 파이프라인 구축

Prometheus 하나만으로도 모니터링은 된다. 근데 주센터와 DR센터에서 Prometheus를 따로 운영하고 있었고, 메트릭을 **한 곳에서 통합 조회**해야 하는 상황이었다.

### 문제

```
주센터  Prometheus  → 각자 따로 운영, 통합 조회 불가
DR센터 Prometheus
```

그리고 각 센터의 Prometheus가 메트릭을 수집하려면 대상 서버에 **Node Exporter가 먼저 설치**되어 있어야 한다. Node Exporter가 없으면 Prometheus는 그 서버의 메트릭을 가져올 수 없다.

```
각 센터 서버
    └── Node Exporter 설치 (서버마다)
            ↓
    각 센터 Prometheus가 스크래핑
            ↓
    중앙 Prometheus가 Federation으로 수집
```

즉 실무에서 파이프라인 구축의 실제 순서는 이렇다:

```
1. 각 서버에 Node Exporter 설치
2. 각 센터 Prometheus가 Node Exporter 스크래핑하도록 설정
3. 중앙 Prometheus가 Federation으로 각 센터 Prometheus 수집
4. Metricbeat → Kafka → Logstash → Elasticsearch로 전달
```

### 구조

```
각 센터 Prometheus → 중앙 Prometheus (Federation) → Metricbeat
                                                        ↓
                                                      Kafka
                                                        ↓
                                            Logstash → Elasticsearch
                                                        ↓
                                                      Kibana
```

### 각 구성 요소 역할 요약

| 구성 요소 | 역할 |
|---|---|
| Node Exporter | 서버 메트릭을 `/metrics`로 노출 |
| 각 센터 Prometheus | Node Exporter 스크래핑, 메트릭 수집 |
| 중앙 Prometheus | Federation으로 각 센터 Prometheus 통합 수집 |
| Metricbeat | 중앙 Prometheus 메트릭을 Kafka로 전달 |
| Kafka | 메트릭 버퍼링, 유실 방지 |
| Logstash | Kafka 구독, 포맷 변환 후 Elasticsearch 전달 |
| Elasticsearch | 메트릭 장기 저장 및 검색 |
| Kibana | 시각화·대시보드 |

---

## 정리

| 개념 | 한 줄 요약 |
|---|---|
| Prometheus | 메트릭을 Pull 방식으로 수집·저장하는 모니터링 도구 |
| 메트릭 | 시스템 상태를 나타내는 숫자값 (CPU, 메모리, 요청 수 등) |
| Pull 방식 | Prometheus가 직접 앱의 /metrics를 주기적으로 가져감 |
| Exporter | 외부 시스템 메트릭을 Prometheus 포맷으로 변환해 노출 |
| PromQL | 수집된 메트릭을 조회하는 쿼리 언어 |
| Federation | 여러 Prometheus의 메트릭을 중앙 Prometheus로 통합 |

> Prometheus는 "지금 이 시스템이 어떤 상태인가"를 숫자로 계속 기록하는 도구다. 혼자 써도 되고, 필요에 따라 Kafka·ES 같은 파이프라인에 연결해 확장할 수 있다.
