# clock

CPU 내부의 모든 동작이 맞추는 박자 신호. CPU 안의 진동자(oscillator)가 1초에 수십억 번 신호를 내보내고, 이 신호 하나가 1 clock cycle이다. "cycle"은 "clock cycle"의 줄임말로, 둘은 같은 뜻이다.

```
3GHz CPU = 1초에 30억 clock = 1 clock ≈ 0.33 나노초
```

CPU는 이 신호에 맞춰 순서대로 동작한다. 덧셈, 비교, 레지스터 이동 같은 단순 연산은 1~2 clock에 끝난다. 메모리 접근처럼 외부 장치가 개입하는 연산은 수십~수백 clock이 걸린다.

clock은 CPU 성능을 이야기할 때 기준 단위가 된다. "이 연산이 몇 clock이냐"는 곧 CPU가 같은 시간에 다른 걸 얼마나 더 할 수 있었느냐와 같은 말이다.

## 관련 개념

- 메모리 접근 비용을 clock으로 비교하면 cache.md, storage-hierarchy.md 참고
