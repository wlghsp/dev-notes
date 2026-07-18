# Kafka acks (Acknowledgement)

프로듀서가 메시지를 보낸 뒤 브로커로부터 "받았다"는 응답을 얼마나 기다릴지 설정하는 값.

`acks` 설정 하나로 프로듀서 → 브로커 구간의 전달 보장 수준이 결정된다.

## acks=0

브로커 응답을 기다리지 않는다. 보내고 끝이다.

가장 빠르지만 유실 가능성이 있다. 브로커가 메시지를 받기 전에 죽으면 아무도 모른다. At-Most-Once에 해당한다.

## acks=1

리더 브로커가 메시지를 받으면 OK 응답을 돌려준다.

리더가 팔로워에게 복제하기 전에 죽으면 메시지가 유실될 수 있다. 응답이 왔다고 해서 완전히 안전한 건 아니다.

## acks=all (또는 acks=-1)

ISR(In-Sync Replica)에 속한 모든 복제본이 메시지를 받아야 OK 응답을 돌려준다.

가장 강한 내구성 보장이다. 리더가 죽어도 팔로워 중 하나가 리더가 되면서 메시지를 이어받을 수 있다. `min.insync.replicas` 설정으로 최소 ISR 수를 지정해야 의미가 있다.

## acks=all만으로는 Exactly-Once가 안 된다

응답이 네트워크에서 유실되면 프로듀서는 실패로 판단하고 재전송한다. 브로커는 이미 받았지만 프로듀서는 모른다. 결과적으로 같은 메시지가 두 번 들어온다.

이 중복을 막으려면 Idempotent Producer가 필요하다.

참고: idempotent-producer.md
참고: at-least-once.md
참고: exactly-once.md
참고: kafka-isr.md
