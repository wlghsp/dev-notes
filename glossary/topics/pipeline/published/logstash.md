# Logstash

Logstash는 데이터를 수집하고 가공해서 Elasticsearch 같은 목적지로 보내주는 데이터 처리 파이프라인 도구다. Elastic Stack의 구성 요소 중 하나다.

## Input → Filter → Output 구조

Logstash 파이프라인은 input(수집) → filter(가공) → output(전달) 세 단계로 이루어진다.

Input은 데이터를 어디서 받을지 정의한다. 파일, Kafka, Beats, HTTP 등 다양한 입력을 지원한다.

Filter는 받은 데이터를 구조화하는 단계다. 정형화되지 않은 텍스트 로그를 파싱해서 필드로 나누는 Grok 필터가 대표적이다.

Output은 가공된 데이터를 보낼 목적지를 정의한다. Elasticsearch가 가장 흔한 목적지이고, 그 외에도 파일이나 S3 등으로 보낼 수 있다.

Input에서 Filter로, Filter에서 Output으로 넘어가는 사이에는 메모리 기반의 큐가 있어서 이벤트를 버퍼링한다.
