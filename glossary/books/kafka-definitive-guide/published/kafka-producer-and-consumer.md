# Kafka Producer와 Consumer

Kafka 클러스터를 사용하는 두 가지 기본 클라이언트 유형.

Kafka Connect나 Kafka Streams 같은 상위 API도 결국 이 둘을 기본 부품으로 삼아 만들어진다.

## Producer

새 메시지를 만들어서 Kafka에 쓰는 쪽이다. 다른 pub/sub 시스템에서는 publisher나 writer라고 부르기도 한다.

프로듀서는 특정 토픽에 메시지를 보낸다. 기본적으로는 토픽의 여러 파티션에 메시지를 고르게 분산시키지만, key를 이용해 특정 파티션으로 몰아서 보낼 수도 있다(참고: kafka-message-and-batch.md).

## Consumer

메시지를 읽는 쪽이다. 다른 pub/sub 시스템에서는 subscriber나 reader라고 부르기도 한다.

컨슈머는 하나 이상의 토픽을 구독하고, 각 파티션에 쓰인 순서 그대로 메시지를 읽는다. 어디까지 읽었는지는 offset이라는 값으로 추적한다.

## Offset

파티션 안에서 메시지 하나하나에 붙는 값으로, 계속 증가한다. 같은 파티션 안에서 다음 메시지는 항상 이전 메시지보다 큰 offset을 가진다.

컨슈머는 다음에 읽어야 할 offset을 Kafka 자체에 저장해둔다. 그래서 컨슈머가 중간에 멈췄다가 다시 시작해도, 이전에 어디까지 읽었는지 잃어버리지 않는다.

참고: consumer-group.md
