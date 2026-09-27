# Signal

OpenTelemetry에서 관측 데이터의 한 종류를 부르는 용어. Trace, Metric, Log 세 가지가 있다.

세 signal은 형태와 쓰임이 다르다.

- Trace: 요청 하나의 전체 경로. 여러 컴포넌트를 거치며 생긴 span들이 모여 하나의 trace를 이룬다. "이 요청이 어디서 느려졌는가"에 답한다.
- Metric: 시간에 따라 집계된 수치. 요청 수, 에러율, latency 분포 같은 것. "지금 시스템 상태가 전반적으로 어떤가"에 답한다.
- Log: 특정 시점에 발생한, 서로 이어지지 않고 독립적으로 툭툭 남는 개별 사건 기록. "그 순간 정확히 무슨 일이 있었는가"에 답한다.

## 왜 하나의 이름으로 묶었는가

OpenTelemetry 이전에는 Trace는 Jaeger/Zipkin, Metric은 Prometheus, Log는 각자 다른 로깅 라이브러리처럼 도구마다 따로 다뤘다. 세 데이터가 서로 다른 포맷, 다른 수집 경로를 가지고 있어서 연결하기가 어려웠다.

OpenTelemetry는 이 셋을 signal이라는 공통 개념으로 묶어서, 같은 API/SDK/Collector 파이프라인으로 다룬다. 세 signal이 공통된 방식으로 수집되기 때문에 서로 연결하기가 쉬워진다. 예를 들어 trace의 span과 log 이벤트를 같은 trace id로 묶어두면, 로그를 보다가 바로 관련된 trace로 넘어갈 수 있다. 이렇게 서로 다른 signal을 연결하는 것이 correlation이다.

## 앞으로 추가될 수 있는 signal

OpenTelemetry 스펙에는 Trace/Metric/Log 외에 Profile 같은 signal도 논의되고 있다. signal이라는 이름 자체가 "관측 가능한 신호의 한 종류"라는 뜻이라, 새로운 종류의 관측 데이터가 생기면 네 번째, 다섯 번째 signal로 편입될 수 있는 구조다.
