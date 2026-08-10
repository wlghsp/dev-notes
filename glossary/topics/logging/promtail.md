# Promtail

Promtail은 각 노드/파드의 로그 파일을 읽어서 loki.md에서 다루는 Loki로 전송(push)하는 로그 수집 에이전트다. Loki 스택에서 로그를 만들어내는 쪽(예: 애플리케이션 컨테이너)과 로그를 저장하는 쪽(Loki) 사이를 잇는 역할을 한다.

## 하는 일

호스트나 Kubernetes 노드에 데몬셋 형태로 떠서 로그 파일을 tail하듯 읽고, 각 로그 라인에 `namespace`, `pod`, `app` 같은 레이블을 붙여 Loki로 보낸다. 이 레이블이 Loki가 로그 본문 대신 인덱싱하는 대상이 되므로, Promtail의 레이블링 설정(scrape config)이 이후 LogQL 쿼리에서 로그를 얼마나 빠르고 정확하게 좁힐 수 있는지를 결정한다.

## Grafana Alloy로의 대체

Promtail과 같은 역할(로그를 읽어서 Loki로 push)을 하는 후속 도구로 alloy.md에서 다루는 Grafana Alloy가 있다. Alloy는 로그뿐 아니라 메트릭까지 하나의 에이전트로 처리하며, Grafana Labs가 신규 구성에 권장하는 표준 수집 에이전트다. Promtail은 유지보수 모드로 들어갔고, 새로 스택을 구성할 때는 Promtail 대신 Alloy를 쓰는 게 현재 방향이다.

```mermaid
graph LR
    App["애플리케이션 / 컨테이너 로그"]
    Promtail["Promtail"]
    Loki["Loki"]
    App -->|로그 파일 tail| Promtail
    Promtail -->|레이블 붙여 push| Loki
```

참고: loki.md, alloy.md
