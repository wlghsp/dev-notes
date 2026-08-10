# Loki

Loki는 Grafana Labs가 만든 로그 수집·저장 시스템이다. Prometheus가 메트릭(숫자 시계열)을 다루듯, Loki는 로그(텍스트)를 같은 방식으로 다룬다 — "로그를 위한 Prometheus"라는 비유가 정확하다.

## 로그 본문 대신 레이블을 인덱싱한다

Elasticsearch처럼 로그 본문 전체에 풀텍스트 인덱스를 만들면 저장 비용과 인덱스 크기가 커진다. Loki는 로그에 붙은 레이블(예: `namespace`, `pod`, `app` 같은 메타데이터)만 인덱싱하고, 그 레이블로 로그 스트림을 좁힌 다음 안에서 grep 방식으로 검색한다. 그래서 운영 비용이 훨씬 싸고, Kubernetes처럼 파드 레이블 체계가 이미 갖춰진 환경에 자연스럽게 맞는다.

## Grafana와의 관계

Loki는 저장·쿼리 엔진이고 Grafana는 그걸 조회하는 시각화 UI다. Grafana 대시보드에 Loki를 데이터소스로 등록하면 LogQL(PromQL과 문법이 유사한 Loki의 쿼리 언어)로 로그를 검색할 수 있고, 같은 대시보드에서 Prometheus 메트릭 그래프와 로그를 나란히 볼 수 있다. 예를 들어 메트릭 패널에서 CPU 스파이크가 찍힌 시점을 클릭하면 같은 시간대 로그를 Loki 패널에서 바로 확인하는 연계가 Grafana의 핵심 사용 시나리오다.

## 전체 흐름

```mermaid
graph LR
    Promtail["Promtail / Grafana Agent"]
    Loki["Loki (저장 + LogQL 쿼리)"]
    Grafana["Grafana"]
    Promtail -->|로그 push| Loki
    Loki -->|조회| Grafana
```

각 노드/파드의 로그 파일은 promtail.md에서 다루는 Promtail이나 alloy.md에서 다루는 Grafana Alloy가 읽어서 Loki로 전송(push)한다. Loki가 이를 레이블 기준으로 저장하고, Grafana가 조회한다. Prometheus(메트릭 수집) + Loki(로그 수집) + Grafana(통합 조회/시각화)가 하나의 관찰성(observability) 스택으로 같이 쓰이는 게 표준적인 패턴이다.

참고: promtail.md, alloy.md
