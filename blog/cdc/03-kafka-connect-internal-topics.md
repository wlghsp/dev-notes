# Kafka Connect 내부 토픽과 cleanup.policy

## Kafka Connect가 내부적으로 쓰는 토픽

Kafka Connect는 자신의 상태를 Kafka 토픽에 저장한다. 세 개의 내부 토픽이 있다.

- `connect-offsets` — 각 커넥터가 어디까지 읽었는지 오프셋 저장
- `connect-config` — 등록된 커넥터 설정 저장
- `connect-status` — 커넥터/태스크 상태 저장

이 토픽들은 Kafka Connect가 시작할 때 자동으로 생성된다. 문제는 **자동 생성 시 기본값이 잘못 설정된다**는 것이다.

---

## cleanup.policy가 왜 중요한가

Kafka 토픽의 `cleanup.policy`는 두 가지다:

- `delete` — 오래된 메시지를 시간/용량 기준으로 삭제
- `compact` — 동일한 key의 메시지 중 최신 것만 유지

Connect 내부 토픽은 반드시 `compact`여야 한다. 오프셋이나 설정 정보가 `delete`로 날아가버리면 커넥터가 어디서부터 다시 읽어야 할지 모른다.

---

## 실제로 겪은 문제

Kafka Connect가 시작하자마자 반복 종료되는 현상이 있었다. 로그를 보니 이런 에러였다:

```
Topic 'connect-offsets' is required to have 'cleanup.policy=compact'
but found 'cleanup.policy=delete'
```

`kafka-configs.sh`로 변경을 시도했는데 반영이 안 됐다. 결국 토픽을 삭제하고 재생성하는 게 확실한 방법이었다.

---

## 올바른 토픽 설정

```bash
# connect-offsets
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic connect-offsets \
  --partitions 25 --replication-factor 3 \
  --config cleanup.policy=compact

# connect-config
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic connect-config \
  --partitions 1 --replication-factor 3 \
  --config cleanup.policy=compact

# connect-status
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic connect-status \
  --partitions 5 --replication-factor 3 \
  --config cleanup.policy=compact
```

파티션 수는 Connect 설정(`worker.config`)의 값과 일치해야 한다.

---

## 주의사항

**Kafka Connect가 실행 중인 상태에서 토픽을 삭제하면 바로 재생성된다.** 반드시 Connect를 먼저 멈추고 삭제해야 한다.

그리고 `connect-offsets`를 초기화하면 **커넥터의 오프셋도 사라진다**. 재등록 시 `snapshot.mode`에 따라 처음부터 스냅샷을 찍거나 최신 binlog부터 시작한다.

---

## 다음

[04 - DELETE 이벤트 처리: tombstone과 rewrite 모드](04-delete-event-handling.md)
