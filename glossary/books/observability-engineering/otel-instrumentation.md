# OpenTelemetry Instrumentation

Instrumentation은 애플리케이션 코드가 실행되는 동안 telemetry(구조화된 이벤트, trace span 등)를 만들어내도록 코드에 장치를 심는 작업이다. OpenTelemetry(OTel)는 이 작업을 벤더에 종속되지 않는 방식으로 할 수 있게 해주는 오픈소스 표준이다.

OTel 이전에는 계측 방식이 벤더마다 제각각이었다. 특정 백엔드용 라이브러리를 붙이면 그 벤더에 종속됐고, 다른 도구로 옮기려면 계측 코드를 처음부터 다시 짜야 했다. OpenTracing(CNCF)과 OpenCensus(Google)가 각각 이 문제를 풀려던 경쟁 표준이었는데, 2019년 두 프로젝트가 합쳐져 OpenTelemetry가 됐다.

## OTel의 핵심 구성 요소

- API: 코드에 계측을 추가할 수 있게 해주는 명세 부분. 실제 구현이 무엇이든 신경 쓰지 않고 계측 코드를 작성할 수 있게 해준다.
- SDK: API의 실제 구현체. 상태를 추적하고 데이터를 배치로 모아 전송을 준비한다.
- Tracer: SDK 안에서, 현재 어떤 span이 활성 상태인지 추적하는 컴포넌트. span을 시작하고, 속성을 붙이고, 종료하는 역할을 한다.
- Meter: SDK 안에서, 어떤 metric들이 보고 가능한지 추적하는 컴포넌트.
- Context propagation: 현재 요청의 context(어떤 trace/span에 속해 있는지)를 W3C Trace Context나 B3 같은 형식으로 직렬화/역직렬화해서 서비스 경계를 넘나들며 전달하는 부분.
- Exporter: SDK가 들고 있는 메모리 상의 span/metric 객체를 실제 백엔드가 이해하는 형식으로 바꿔 내보내는 플러그인. 로컬 파일일 수도, OpenTelemetry Collector일 수도, Jaeger/Honeycomb 같은 원격 백엔드일 수도 있다.
- Collector: 독립 실행되는 프로세스로, telemetry 데이터를 받아서 처리한 뒤 하나 이상의 목적지로 전달하는 프록시/사이드카 역할을 한다.

## Automatic Instrumentation과 Custom Instrumentation

Automatic instrumentation은 애플리케이션 코드를 거의 건드리지 않고, OTel이 제공하는 wrapper/interceptor/agent를 붙이는 것만으로 HTTP나 gRPC, DB 호출 같은 표준적인 지점에 자동으로 span을 생성하게 만드는 방식이다. 어떤 서비스가 어떤 서비스를 부르는지에 대한 뼈대를 빠르게 얻을 수 있다.

Custom instrumentation은 여기에 비즈니스 로직 관점에서 의미 있는 필드(고객 ID, 처리한 항목 수, 에러 원인 등)를 직접 span에 추가하는 작업이다. Automatic instrumentation만으로는 "무엇이 느린지"는 보여줘도 "왜 느린지"까지는 잘 보여주지 못하는 경우가 많아서, 실제로 유용한 observability는 custom instrumentation을 더했을 때 나온다.

## 데이터를 백엔드로 보내는 두 경로

계측된 데이터를 최종적으로 내보내는 방법은 두 가지다. 프로세스에서 백엔드로 직접 export하거나, OpenTelemetry Collector를 거쳐서 보내는 것. Collector를 거치면 여러 백엔드로 동시에 데이터를 보내거나, 데이터를 가공/필터링하는 작업을 애플리케이션 코드와 분리할 수 있다.

참고: structured-event.md, trace-span.md, signal.md

## Recap

OpenTelemetry는 벤더에 종속되지 않는 계측 표준으로, API/SDK/Tracer/Meter/Context propagation/Exporter/Collector 구성 요소가 서로 맞물려 동작한다. Automatic instrumentation으로 서비스 간 호출의 뼈대를 빠르게 얻고, Custom instrumentation으로 비즈니스 로직 관점의 필드를 더해야 실제로 쓸모 있는 observability가 나온다. 계측된 데이터는 직접 export하거나 Collector를 거쳐 백엔드로 보낼 수 있다.
