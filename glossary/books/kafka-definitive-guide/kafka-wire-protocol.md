# Kafka Wire Protocol

브로커와 클라이언트(producer, consumer)가 TCP 연결 위에서 주고받는, Kafka가 직접 정의한 바이너리 요청/응답 규약.

HTTP나 REST가 아니라 Kafka 고유의 바이트 포맷을 쓴다. 이 설계를 택한 이유는 kafka-vs-websocket.md에서 다룬다.

## 요청/응답 한 쌍의 기본 구조

클라이언트가 보내는 모든 요청은 크게 두 부분으로 나뉜다. 요청이 어떤 종류인지, 어떤 버전인지, 어떤 클라이언트가 보냈는지를 나타내는 헤더가 앞에 오고, 그 뒤에 요청 종류별로 정해진 실제 데이터(바디)가 따라온다.

브로커는 이 요청을 받아 처리한 뒤, 같은 형식의 응답을 돌려준다. 요청과 응답 모두 맨 앞에 전체 메시지 길이를 담은 4바이트가 붙어서, 수신 측이 몇 바이트를 더 읽어야 하나의 메시지가 끝나는지 알 수 있다.

## API key와 API version

Kafka 프로토콜은 요청 종류마다 고유 번호(API key)를 부여한다. 예를 들어 Produce는 0번, Fetch는 1번, Metadata는 3번이다.

같은 API key라도 여러 버전(API version)이 동시에 존재할 수 있다. Kafka가 새 기능을 추가하면서 요청/응답 포맷을 바꿔야 할 때, 기존 버전을 없애지 않고 새 버전을 추가하는 방식으로 하위 호환성을 유지한다. 클라이언트와 브로커는 연결 초기에 서로 지원하는 버전 범위를 확인(ApiVersionsRequest)하고, 둘 다 지원하는 버전으로 통신한다.

## correlation ID로 요청과 응답을 매칭

Kafka 클라이언트는 하나의 TCP 연결 위로 여러 요청을 순차적으로 또는 파이프라인으로 보낼 수 있다. 이때 브로커가 어떤 응답이 어떤 요청에 대한 것인지 구분할 수 있어야 한다.

이를 위해 클라이언트는 요청마다 correlation ID를 붙여 보내고, 브로커는 응답에 같은 correlation ID를 그대로 실어 돌려준다. 클라이언트는 이 값으로 응답을 원래 요청과 매칭한다.

## 대표적인 요청 종류

프로듀서가 메시지를 쓸 때는 ProduceRequest를 보낸다(참고: kafka-producer-and-consumer.md). 컨슈머가 메시지를 읽을 때는 FetchRequest를 보내고, 어디까지 읽었는지는 offset으로 지정한다.

클라이언트가 클러스터 구조(어떤 토픽의 어떤 파티션이 어떤 브로커에 있는지)를 알아야 할 때는 MetadataRequest를 보낸다(참고: kafka-bootstrap-and-advertised-listeners.md).

참고: kafka-vs-websocket.md, pub-sub-messaging.md
