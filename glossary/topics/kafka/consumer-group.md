# Consumer Group

Kafka 토픽을 여러 컨슈머가 나눠서 읽는 단위.

같은 group.id를 가진 컨슈머들이 하나의 그룹을 이룬다. 파티션은 그룹 내 컨슈머에게 하나씩 배정된다. 하나의 파티션은 같은 그룹 내에서 동시에 하나의 컨슈머만 읽는다.

## 파티션과 컨슈머의 관계

```
토픽 (파티션 3개)
  partition-0 → consumer-A
  partition-1 → consumer-B
  partition-2 → consumer-C
```

컨슈머가 파티션보다 많으면 남는 컨슈머는 놀게 된다. 파티션이 컨슈머보다 많으면 한 컨슈머가 여러 파티션을 담당한다.

## Offset 커밋

컨슈머는 자신이 어디까지 읽었는지 offset을 `__consumer_offsets` 토픽에 커밋한다.

컨슈머가 죽고 재시작되거나, 리밸런싱이 일어나 다른 컨슈머가 파티션을 이어받을 때, 마지막으로 커밋된 offset부터 다시 읽기 시작한다.

offset 커밋 시점이 전달 보장을 결정한다.

- 처리 전 커밋 → At-Most-Once. 처리 실패해도 다시 읽지 않음. 유실 가능.
- 처리 후 커밋 → At-Least-Once. 커밋 전 죽으면 재시작 후 같은 메시지를 다시 읽음. 중복 가능.

## 리밸런싱

그룹에 컨슈머가 추가되거나 제거될 때 파티션 배정이 재조정된다.

리밸런싱 중에는 그룹 전체가 일시 정지된다. 컨슈머가 처리 중이던 메시지가 있으면 커밋되지 않은 offset은 다음 컨슈머가 다시 읽게 된다. At-Least-Once 특성이 여기서도 나타난다.

## 독립적인 소비

서로 다른 그룹은 같은 토픽을 독립적으로 읽는다. 각 그룹이 자신의 offset을 따로 관리하기 때문이다.

```
토픽 partition-0
  group-A → offset 100까지 읽음
  group-B → offset 50까지 읽음
```

group-A가 offset을 진행시켜도 group-B에 영향 없다.

참고: offset.md
참고: at-least-once.md
참고: kafka-transactions.md
