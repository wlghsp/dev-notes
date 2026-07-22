# Vector

Vector는 로그와 메트릭을 여러 소스에서 수집해서 원하는 목적지로 전달해주는 관측 가능성(observability) 데이터 파이프라인 도구다. Datadog이 만들고 관리한다.

## Source → Transform → Sink 구조

Vector는 데이터를 source(수집) → transform(변환, 선택) → sink(전달)라는 세 단계로 처리한다.

Source는 데이터를 어디서 가져올지 정의한다. 파일, Prometheus, Datadog 에이전트 등 다양한 소스를 지원한다.

Transform은 가져온 데이터를 가공하는 단계로, 필요할 때만 거친다. Vector는 VRL(Vector Remap Language)이라는 자체 언어로 파싱, 필드 추가, 필터링 같은 변환을 수행한다.

Sink는 가공된 데이터를 어디로 보낼지 정의한다. Elasticsearch, Kafka, S3, Prometheus 등 다양한 목적지로 동시에 보낼 수 있다.

## 왜 쓰는가

같은 역할을 하는 Logstash나 Fluentd 대비 훨씬 적은 메모리를 쓰고, 하나의 정적 바이너리로 배포된다는 게 특징이다. Rust로 작성되어 있다.

## 커뮤니케이션에서 헷갈리는 지점

로그 전용 도구로 오해하기 쉽지만, Vector가 다루는 데이터 단위(Event)는 로그와 메트릭을 모두 포함한다. 그래서 지호님 파이프라인처럼 Prometheus에서 메트릭을 가져와 Kafka로 넘기는 것도 Vector의 정상적인 용도다.

참고: prometheus.md, logstash.md
