# Kafka는 왜 WebSocket을 쓰지 않는가

Kafka는 브로커와 클라이언트(producer, consumer) 간 통신에 자체 바이너리 프로토콜(참고: kafka-wire-protocol.md)을 쓴다. WebSocket 같은 범용 메시징 프로토콜을 쓰지 않고 왜 굳이 자체 프로토콜을 설계했는지가 이 문서의 주제다.

## 이유 1: zero-copy 전송을 깨지 않기 위해

Kafka는 컨슈머에게 데이터를 보낼 때 zero-copy 전송(sendfile 시스템 콜)을 쓴다. 디스크 페이지 캐시에 있는 데이터를 커널 레벨에서 바로 네트워크 소켓으로 흘려보내는 방식이라, 애플리케이션 레벨로 데이터를 복사하는 과정이 생략된다.

이 zero-copy 경로는 Kafka가 대용량 처리량을 낼 수 있는 핵심 기법 중 하나다. 그런데 WebSocket처럼 애플리케이션 레벨에서 프레이밍을 해야 하는 프로토콜을 쓰면, 커널이 데이터를 그대로 소켓에 흘려보낼 수 없게 되어 이 경로가 깨진다.

## 이유 2: Kafka 의미론에 맞춘 설계

배치 전송, 압축(gzip/snappy/lz4/zstd), 오프셋 기반 fetch, 파티션별 acks(0/1/all) 같은 기능들이 전부 이 프로토콜 안에 최적화되어 들어가 있다(참고: kafka-message-and-batch.md).

범용 메시징 프로토콜 위에 이런 기능을 얹으려면 결국 그 위에 또 다른 프로토콜을 하나 더 정의해야 하는 셈이라, 이중 오버헤드가 생긴다.

## 이유 3: 애초에 설계 목적이 다르다

WebSocket은 브라우저가 raw TCP 소켓을 열 수 없다는 제약 때문에 만들어졌다. HTTP 핸드셰이크로 시작해서 서버-브라우저 간 양방향 통신을 가능하게 하는 것이 목적이라, 브라우저 호환성과 방화벽/프록시 통과가 주된 관심사다.

반면 Kafka 클라이언트는 대부분 백엔드 서비스이지 브라우저가 아니다. 브라우저 호환성이 필요 없는 환경에서 HTTP 업그레이드 핸드셰이크나 WebSocket 프레이밍 오버헤드를 감수할 이유가 없다.

## 정리

WebSocket은 브라우저도 서버처럼 양방향 통신을 하게 해주는 범용 도구이고, Kafka 프로토콜은 브로커-클라이언트 간 초고처리량 스트리밍에 특화된 전용 도구다. 목적이 다르니 최적화 지점도 다르다.

## 브라우저에서 Kafka 데이터를 실시간으로 봐야 한다면

Kafka 프로토콜 자체를 WebSocket으로 바꾸는 게 아니라, Kafka 앞단에 브릿지/프록시를 둔다.

이 프록시가 Kafka 프로토콜로 브로커와 통신하고, 브라우저 쪽에는 WebSocket으로 중계한다. Confluent REST Proxy는 이 목적으로 흔히 언급되지만 실제로는 HTTP(polling) 기반이라 WebSocket 중계 자체는 하지 않으며, 브라우저에 진짜 실시간 push가 필요하면 별도의 커스텀 WebSocket 게이트웨이 서비스를 둬야 한다. 즉 Kafka가 WebSocket을 쓰는 게 아니라 WebSocket이 Kafka를 감싸는 구조다.

참고: pub-sub-messaging.md, kafka-bootstrap-and-advertised-listeners.md
