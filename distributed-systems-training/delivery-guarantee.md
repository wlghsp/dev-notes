# Delivery Guarantee (전달 보장)

메시징 시스템이 "메시지가 몇 번, 어떤 순서로 소비자에게 전달되는가"에 대해 제공하는 보장 수준.

Kafka, RabbitMQ, SQS는 이 보장을 서로 다른 방식으로 구현한다.

## 세 가지 수준

**At-most-once**
메시지가 전달되지 않을 수 있지만, 중복은 절대 없다. consumer가 메시지를 받기 전에 broker가 이미 "전달 완료"로 표시하는 방식일 때 발생한다. 유실을 감수해도 되는 로그 수집 같은 곳에 쓴다.

**At-least-once**
메시지가 반드시 전달되지만, 중복될 수 있다. consumer가 메시지를 처리하고 ack를 보내기 전에 죽으면, broker는 ack를 못 받았으니 다시 보낸다. 대부분의 메시징 시스템 기본값이다.

**Exactly-once**
메시지가 정확히 한 번만 처리된 것처럼 보이게 만든다. 네트워크 계층에서 완벽히 구현하기는 사실상 불가능하고, 보통 at-least-once 전달 + 수신 측 멱등 처리(idempotency.md)를 결합해서 흉내낸다.

## 왜 완벽한 exactly-once가 어려운가

producer가 메시지를 보내고 broker의 ack를 못 받으면, producer는 "broker가 못 받았나?"와 "broker는 받았는데 ack만 유실됐나?"를 구분할 수 없다. 안전하게 하려면 재전송해야 하는데, 그러면 broker가 이미 받은 메시지가 중복 저장될 수 있다. 이 딜레마 때문에 순수 전달 계층만으로는 exactly-once를 보장할 수 없고, 반드시 수신 측의 멱등성이나 트랜잭션이 결합돼야 한다.

## 실제 시스템 비교

**Kafka**
기본은 at-least-once. producer에 `idempotent producer` 옵션을 켜면 같은 메시지의 중복 전송을 broker가 자동으로 걸러낸다. 여기에 `transactional producer`까지 쓰면 여러 파티션에 걸친 쓰기를 원자적으로 묶어서, producer-to-broker 구간에서는 사실상 exactly-once에 가까운 보장을 제공한다. 다만 consumer가 처리 후 상태를 저장하는 부분까지 포함한 end-to-end exactly-once는 여전히 애플리케이션이 멱등하게 설계해야 한다.

**RabbitMQ**
기본은 at-least-once (consumer ack 기반). Publisher confirm과 consumer ack를 함께 켜야 유실을 최소화할 수 있다. 메시지 순서는 큐 단위로는 보장되지만, Kafka의 파티션 같은 명시적 순서 보장 단위는 없다 — 여러 consumer가 하나의 큐를 나눠 가져가면 순서가 섞일 수 있다.

**Amazon SQS**
Standard 큐는 at-least-once이며 순서를 보장하지 않는다 (성능과 처리량이 우선). FIFO 큐를 선택하면 순서 보장 + 중복 방지(5분 내 중복 제거)를 제공하지만, 처리량이 Standard보다 훨씬 낮다. 즉 SQS는 "순서/중복 보장"과 "처리량" 사이의 트레이드오프를 큐 타입 선택으로 명시적으로 노출한다.

## 순서 보장이 파티션/큐 단위인 이유

세 시스템 모두 "전체 스트림의 순서"를 보장하지는 않는다. Kafka는 파티션 내에서만, SQS FIFO는 메시지 그룹 내에서만 순서를 보장한다. 전체 순서를 보장하려면 병렬 처리를 포기해야 하기 때문에, 대신 "같은 키를 가진 메시지끼리는 같은 파티션/그룹으로 보내서 그 안에서만 순서를 보장"하는 절충안을 택한다.

## 정리

| | 기본 보장 | 순서 보장 단위 | 중복 방지 |
|---|---|---|---|
| Kafka | at-least-once | 파티션 | idempotent/transactional producer |
| RabbitMQ | at-least-once | 큐 (단일 consumer일 때만) | 없음 (직접 구현 필요) |
| SQS Standard | at-least-once | 없음 | 없음 |
| SQS FIFO | at-least-once + 중복 제거 | 메시지 그룹 | 5분 중복 제거 윈도우 |

세 시스템 모두 "완벽한 exactly-once"를 전달 계층만으로 풀지 않는다. 대신 idempotency.md(수신 측 멱등 처리)와 결합해서 실질적인 정확성을 확보한다.

참고: idempotency.md, eventual-consistency.md, stream-processing.md
