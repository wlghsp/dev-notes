# AMQP (Advanced Message Queuing Protocol)

메시지 브로커와 클라이언트 사이의 통신 방식을 정의한 네트워크 프로토콜이다. RabbitMQ가 이 프로토콜을 구현한 대표적인 브로커다.

HTTP처럼 애플리케이션 레이어 프로토콜이다. 어떤 언어로 만든 producer든, 어떤 언어로 만든 consumer든 AMQP를 구현하면 같은 브로커와 통신할 수 있다.

## 핵심 구성 요소

**Exchange**
producer가 메시지를 보내는 곳이다. producer는 큐에 직접 보내지 않고 Exchange에 보낸다. Exchange는 라우팅 규칙에 따라 메시지를 어떤 큐로 보낼지 결정한다.

Exchange 타입이 라우팅 방식을 결정한다.
- Direct: routing key가 정확히 일치하는 큐로 보낸다
- Fanout: 연결된 모든 큐로 보낸다. 브로드캐스트
- Topic: routing key 패턴으로 큐를 고른다. `order.*`, `*.created` 같은 와일드카드 지원
- Headers: 메시지 헤더 값으로 라우팅한다

**Queue**
메시지가 실제로 쌓이는 곳이다. consumer는 큐를 구독해서 메시지를 가져간다. 메시지를 consumer에게 전달하면 큐에서 삭제된다.

**Binding**
Exchange와 Queue를 연결하는 규칙이다. "이 Exchange에서 이 routing key를 가진 메시지는 저 Queue로 보내라"는 설정이다.

**Routing Key**
producer가 메시지를 보낼 때 붙이는 문자열이다. Exchange가 이 값을 보고 어떤 큐로 보낼지 결정한다.

## 메시지 흐름

```
producer → Exchange → (Binding + Routing Key) → Queue → consumer
```

producer는 Exchange 이름과 routing key만 알면 된다. 어떤 큐로 가는지는 몰라도 된다. 라우팅 로직이 브로커 안에 있다.

## Kafka와의 차이

AMQP(RabbitMQ)는 브로커가 라우팅을 책임진다. 복잡한 조건으로 메시지를 분기할 수 있고, 전달하면 큐에서 사라진다.

Kafka는 라우팅 개념이 없다. producer가 토픽과 파티션을 직접 지정한다. 메시지는 읽어도 사라지지 않고 retention 기간 동안 남는다.

라우팅이 복잡하고 "전달하고 끝"이면 AMQP, 이벤트를 보존하고 여러 곳에서 재소비해야 하면 Kafka가 맞는다.

참고: kafka-vs-rabbitmq.md
참고: eda.md
