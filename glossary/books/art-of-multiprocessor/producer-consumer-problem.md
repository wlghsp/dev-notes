# Producer-Consumer 문제

참고: The Art of Multiprocessor Programming — Herlihy & Shavit (Morgan Kaufmann, 2012), Ch.1

## 문제 설정

mutual-exclusion-problem.md의 Alice와 Bob 우화가 이어진다. 둘은 이혼했고, Alice가 애완동물을 맡게 되어 Bob은 마당에 들어가지 않으면서 먹이만 넣어줘야 한다. 조건이 하나 더 붙는다. Alice는 먹이가 없는데 애완동물을 풀어주고 싶지 않고, Bob은 애완동물이 먹이를 다 먹기 전에는 마당에 들어가고 싶지 않다. 이렇게 한쪽(생산자)이 뭔가를 만들어 놓으면 다른 쪽(소비자)이 그것을 사용하는 구조의 문제를 producer-consumer 문제라고 한다.

## 깡통-실 프로토콜의 재활용

흥미롭게도, mutual exclusion에는 부적합했던 깡통-실 프로토콜이 producer-consumer 문제에는 정확히 들어맞는다. Bob이 깡통을 세워두고(먹이 없음 상태) 마당에 먹이를 놓은 뒤 깡통을 넘어뜨린다(먹이 있음 상태로 전환). Alice는 깡통이 넘어져 있을 때만 애완동물을 풀어주고, 애완동물이 먹이를 다 먹으면 깡통을 다시 세운다.

깡통의 상태(서 있음/넘어짐)가 곧 마당의 상태를 그대로 반영한다. 깡통이 넘어져 있으면 먹이가 있다는 뜻이고, 서 있으면 먹이가 없어 Bob이 더 넣어줄 수 있다는 뜻이다.

## 요구되는 세 가지 성질

1. **Mutual Exclusion**: Bob과 애완동물이 마당에 동시에 있지 않는다.
2. **Starvation-freedom**: Bob이 항상 기꺼이 먹이를 주고 애완동물이 항상 배고파 있다면, 애완동물은 무한히 자주 먹을 수 있어야 한다.
3. **Producer-Consumer**: 먹이가 없으면 애완동물이 마당에 들어가지 않고, 먹지 않은 먹이가 남아 있으면 Bob이 더 넣지 않는다.

## Mutual Exclusion 문제와의 차이

같은 깡통-실 프로토콜이지만, mutual exclusion에 적용했을 때와 producer-consumer에 적용했을 때 요구되는 보장이 다르다는 점이 중요하다.

Mutual exclusion은 deadlock-freedom을 요구한다. 상대가 마당에 없어도, 누구든 혼자서 무한히 자주 마당에 들어갈 수 있어야 한다.

Producer-consumer의 starvation-freedom은 반대로 양쪽의 지속적인 협력을 전제한다. Bob이 먹이 주기를 멈추면 애완동물은 당연히 굶는다. 즉 이 프로토콜은 한쪽이 계속 협조한다는 가정 아래서만 성립하는 더 약한 보장이다.

## 여기서도 등장하는 대기(Waiting)

이 프로토콜도 mutual-exclusion-problem.md에서 다룬 대기 문제에서 자유롭지 않다. Bob이 마당에 먹이를 놓아두고 깡통을 넘어뜨리는 것을 깜빡한 채 휴가를 가버리면, 먹이가 있는데도 애완동물이 굶주릴 수 있다.

## 실무 연결

producer-consumer 문제는 병렬/분산 시스템 거의 전반에서 나타난다. 프로세서들이 네트워크나 공유 버스를 통해 데이터를 주고받을 때, 통신 버퍼에 데이터를 놓는 쪽과 읽어가는 쪽의 관계가 정확히 이 구조다.

---

참고: mutual-exclusion-problem.md, readers-writers-problem.md
