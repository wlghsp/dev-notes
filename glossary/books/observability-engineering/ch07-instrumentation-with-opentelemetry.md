# Chapter 7 — Instrumentation with OpenTelemetry

이 챕터에서 생성된 키워드 파일: otel-instrumentation.md

---

## 이 챕터가 답하는 질문

6장에서 trace를 손으로(직접 map을 채우고 헤더를 실어 보내며) 만들어봤다. 실무에서는 이걸 매번 손으로 하지 않는다. 이 챕터는 그 반복 작업을 대신해주는 표준이 왜 필요했고, OpenTelemetry가 그걸 어떻게 해결하는지를 다룬다.

## 계측이 벤더에 종속됐던 문제

과거에는 계측 라이브러리가 백엔드마다 따로 있었다. 어떤 모니터링 솔루션을 쓰느냐에 따라 계측 코드 자체를 다르게 짜야 했고, 도구를 바꾸려면 계측을 처음부터 다시 해야 했다. 이 문제를 풀기 위해 OpenTracing(CNCF)과 OpenCensus(Google)가 각자 등장했다가, 2019년 두 프로젝트가 합쳐지며 OpenTelemetry가 됐다. 지금은 계측을 한 번만 하고 백엔드는 자유롭게 바꿀 수 있는 게 OTel의 핵심 가치다.

## OTel의 부품들이 서로 어떻게 맞물리는가

API는 계측 코드를 작성하기 위한 명세이고, SDK는 그 명세의 실제 구현체다. SDK 안에서 Tracer는 지금 어떤 span이 활성 상태인지 추적하고, Meter는 어떤 metric이 보고 가능한지 추적한다. Context propagation은 현재 요청이 어떤 trace/span에 속하는지를 W3C Trace Context 같은 형식으로 직렬화해서 서비스 경계 너머로 전달하는 역할을 한다. Exporter는 메모리에 있는 span/metric 객체를 실제 백엔드가 이해하는 형식으로 바꿔 내보내고, Collector는 이 telemetry를 받아 가공한 뒤 하나 이상의 목적지로 전달하는 독립 프로세스다.

## Automatic Instrumentation부터 시작하는 이유

분산 tracing을 도입할 때 제일 큰 난관은, 계측 코드를 어디서부터 추가해야 할지 감이 안 잡힌다는 점이다. OTel은 HTTP, gRPC, DB 호출처럼 흔한 지점에 대해 wrapper/interceptor를 붙이는 것만으로 자동으로 span을 만들어주는 automatic instrumentation을 제공한다. 이걸로 "누가 누구를 부르는가"에 대한 뼈대를 빠르게 얻을 수 있다.

하지만 automatic instrumentation만으로는 "무엇이 느린지"까지는 보여도 "왜 느린지"는 잘 안 보인다. 여기에 비즈니스 로직 관점에서 의미 있는 필드(고객 ID, 처리한 데이터 양, 에러 원인 등)를 custom instrumentation으로 덧붙여야 실제로 쓸모 있는 observability가 나온다.

## 이 챕터가 스터디 2회차(Zero-Code Instrumentation)와 맞닿는 지점

스터디 2회차는 Java Agent/Spring Boot Starter를 통한 "Zero-Code" 계측을 다룬다. 이 챕터에서 말하는 automatic instrumentation과 정확히 같은 개념이다. 다만 이 책의 예제는 Go 기준으로 명시적 wrapper를 붙이는 방식이고, Java에서는 실행 시점에 바이트코드를 조작해 자동으로 계측을 주입하는 Java Agent 방식을 주로 쓴다는 차이가 있다 — 이 차이는 스터디 2회차에서 직접 다뤄질 부분이다.

## 다음으로 이어지는 질문

이 챕터의 마지막 문장은 "metric은 왜 여전히 '필요하다면'이라는 단서가 붙는가"이고, 이건 8~9장(이 로드맵에서는 우선순위가 낮게 잡힌 범위)에서 다룬다.
