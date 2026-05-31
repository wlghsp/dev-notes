# oom-killer

메모리가 부족할 때 OS가 프로세스를 강제 종료해 메모리를 확보하는 메커니즘. OOM은 Out Of Memory의 약자다.

## 왜 메모리가 부족해지는가

각 프로세스는 가상 주소 공간을 통해 메모리 전체를 혼자 쓰는 것처럼 동작한다. 하지만 실제 물리 DRAM은 유한하고, 여러 프로세스가 하나의 DRAM을 나눠 쓴다. 모든 프로세스의 활성 페이지 합이 DRAM 용량을 초과하는 순간 물리 메모리가 부족해진다.

## swap과의 관계

DRAM이 꽉 차면 OS는 바로 OOM Killer를 부르지 않는다. 먼저 swap을 완충재로 쓴다. 당장 안 쓰는 페이지를 디스크(swap)로 내려서 DRAM 빈 자리를 만든다. swap이 있는 동안은 OOM Killer가 개입하지 않는다. swap마저 꽉 차면 그때 비로소 내릴 곳이 없어지고 OOM Killer가 호출된다.

```
DRAM 부족 → swap으로 페이지를 내림 → DRAM 확보
    ↓ (swap도 꽉 참)
더 이상 내릴 곳 없음 → OOM Killer 호출
```

## 핵심 아이디어

RAM도 꽉 차고 swap도 꽉 찼을 때 OS는 더 이상 메모리를 줄 수 없다. 이 상태에서 새 메모리 요청이 들어오면 아무것도 안 하면 시스템 전체가 멈춘다. OOM Killer는 이 상황을 타개하기 위해 희생시킬 프로세스를 골라 강제 종료한다.

## 어떤 프로세스를 죽이는가

Linux는 각 프로세스에 oom_score를 매긴다. 점수가 높을수록 먼저 죽는다.

- 메모리를 많이 쓰는 프로세스일수록 점수가 높다
- 오래 실행된 프로세스는 점수가 낮다 (중요한 프로세스일 가능성이 높다)
- `oom_score_adj` 값으로 수동 조정 가능하다. -1000이면 절대 죽이지 않음, 1000이면 가장 먼저 죽임

## 서버에서 OOM Killer가 발생했는지 확인하는 법

```
dmesg | grep -i "oom"
```

로그에 `Out of memory: Kill process` 메시지가 있으면 OOM Killer가 동작한 것이다. 어떤 프로세스가 얼마만큼의 메모리를 쓰다가 죽었는지도 함께 출력된다.

## Kubernetes와 OOM Killer

Kubernetes는 swap을 끄고 OOM Killer에 의존한다. Pod이 메모리 limit을 초과하면 OOM Killer가 즉시 해당 컨테이너를 죽이고, Kubernetes가 다른 노드에서 재시작한다. 느리게 살아있는 것보다 빠르게 죽고 재시작하는 것이 전체 클러스터 안정성에 낫다.

## 관련 개념

- swap이 꽉 찬 상황과의 관계는 swap.md 참고
- 메모리 부족이 발생하는 근본 구조는 virtual-memory.md 참고
