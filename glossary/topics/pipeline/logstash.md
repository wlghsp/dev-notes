# Logstash

Logstash는 데이터를 수집하고 가공해서 Elasticsearch 같은 목적지로 보내주는 데이터 처리 파이프라인 도구다. Elastic Stack의 구성 요소 중 하나다.

## Input → Filter → Output 구조

Logstash 파이프라인은 input(수집) → filter(가공) → output(전달) 세 단계로 이루어진다.

Input은 데이터를 어디서 받을지 정의한다. 파일, Kafka, Beats, HTTP 등 다양한 입력을 지원한다.

Filter는 받은 데이터를 구조화하는 단계다. 정형화되지 않은 텍스트 로그를 파싱해서 필드로 나누는 Grok 필터가 대표적이다.

Output은 가공된 데이터를 보낼 목적지를 정의한다. Elasticsearch가 가장 흔한 목적지이고, 그 외에도 파일이나 S3 등으로 보낼 수 있다.

Input에서 Filter로, Filter에서 Output으로 넘어가는 사이에는 메모리 기반의 큐가 있어서 이벤트를 버퍼링한다.

## 커뮤니케이션에서 헷갈리는 지점

Vector와 역할이 겹쳐 보이지만(둘 다 수집→가공→전달 구조), Logstash는 JVM 기반이라 Vector보다 메모리를 많이 쓴다. 지호님 파이프라인에서 Vector가 앞단(Prometheus→Kafka)을, Logstash가 뒷단(Kafka→ES)을 맡는 것처럼 역할을 나눠 쓰는 구성도 흔하다.

참고: vector.md
