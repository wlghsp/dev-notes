# Deduplication (중복 제거)

중복으로 들어온 메시지나 요청을 감지하고 무시하는 기법.

수신 측이 멱등하지 않을 때, At-Least-Once 환경에서 Exactly-Once처럼 동작하게 만드는 방법이다.

## 기본 동작 원리

1. 송신자가 메시지에 고유한 ID를 붙여서 보낸다
2. 수신자가 메시지를 처리하고, 처리한 ID를 저장한다
3. 같은 ID가 다시 들어오면 처리하지 않고 무시한다

중복 여부를 판단하는 기준은 이 고유 ID다.

## ID는 누가 만드는가

보통 송신자가 만든다. UUID나 요청의 내용 기반 해시를 쓰기도 한다.

중요한 점은 재전송할 때 **같은 ID를 유지**해야 한다는 것이다. 재시도할 때마다 새 ID를 붙이면 deduplication이 의미가 없다.

## 처리한 ID를 얼마나 오래 보관하는가

영구 보관은 비효율적이다. 보통 유효 시간(TTL)을 설정해서 일정 시간이 지난 ID는 버린다.

TTL이 너무 짧으면, TTL 내에 재전송이 들어와야 deduplication이 동작한다. TTL보다 늦게 재전송이 오면 중복 처리가 일어날 수 있다. 시스템의 재시도 타임아웃보다 TTL이 충분히 길어야 한다.

## Kafka에서의 Deduplication

Kafka의 Idempotent Producer는 producer ID와 시퀀스 번호를 메시지에 붙인다. 브로커는 이미 받은 시퀀스 번호가 들어오면 무시한다. 이 방식으로 프로듀서 → 브로커 구간의 중복을 제거한다.

소비자 측 중복은 소비자가 직접 처리해야 한다. 브로커가 소비자의 처리 결과를 알 수 없기 때문이다.

## Deduplication과 Idempotency의 차이

- Idempotency: 작업 자체를 중복에 강하게 설계하는 것. 수신 측의 성질.
- Deduplication: 중복을 감지하고 걸러내는 메커니즘. 인프라 레이어의 기능.

둘은 목적이 같지만 접근 방식이 다르다. Deduplication은 작업을 변경하지 않고 중복을 차단하고, Idempotency는 중복이 들어와도 결과가 같도록 작업을 설계한다.

참고: at-least-once.md
참고: exactly-once.md
참고: idempotency.md
