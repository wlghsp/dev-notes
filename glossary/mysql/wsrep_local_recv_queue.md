# wsrep_local_recv_queue

Galera Cluster에서 각 노드가 다른 노드로부터 받은 Write-Set을 아직 적용하지 못하고 대기 중인 큐의 길이.

## 의미

Write-Set을 받았다고 해서 바로 스토리지에 적용되는 건 아니다. 노드는 수신한 Write-Set을 이 큐에 쌓아두고 순서대로 처리한다. 큐가 쌓인다는 건 노드가 클러스터의 쓰기 속도를 따라가지 못하고 있다는 뜻이다.

## 확인 방법

```sql
SHOW STATUS LIKE 'wsrep_local_recv_queue%';
```

- `wsrep_local_recv_queue` — 현재 큐 길이. 0이 정상
- `wsrep_local_recv_queue_avg` — 평균값. 지속적으로 높으면 병목 신호

## 해석

- 0~1 → 정상
- 지속적으로 높음 → 이 노드가 클러스터 속도를 소화 못 함. CPU/디스크 병목 의심
- 점점 증가 → Flow Control 발동으로 클러스터 전체 쓰기 속도가 이 노드에 맞춰 떨어짐

참고: wsrep.md
