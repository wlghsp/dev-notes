# Readers-Writers 문제

참고: The Art of Multiprocessor Programming — Herlihy & Shavit (Morgan Kaufmann, 2012), Ch.1

## 문제 설정

Alice와 Bob이 애완동물 근황을 간단한 메시지로 주고받기로 한다. Bob이 집 앞에 큰 게시판을 세워두고, 타일 하나에 글자 하나씩 놓는 방식으로 메시지를 적는다(글쓰기, write). Alice는 망원경으로 게시판을 한 글자씩 읽는다(읽기, read).

## 왜 이 방식이 실패하는가

Bob이 "sell the cat"이라는 메시지를 게시판에 올리는 도중, Alice가 망원경으로 "sell the"까지만 읽었다고 하자. 이 시점에 Bob이 타일을 전부 내리고 "wash the dog"이라는 새 메시지로 바꿔 쓰기 시작한다. Alice가 계속 스캔하면 앞부분은 이전 메시지("sell the"), 뒷부분은 새 메시지("dog")가 섞여서 "sell the dog"이라는 전혀 다른 메시지를 읽게 된다.

문제의 핵심은 읽기와 쓰기가 동시에 일어나면, 읽는 쪽이 일관성 없는 상태(쓰다 만 중간 상태)를 관찰할 수 있다는 것이다.

## 단순한 해결책들

- mutual-exclusion-problem.md의 프로토콜을 써서, Alice가 완전한 문장만 읽도록 강제할 수 있다. 다만 문장 하나를 통째로 놓칠 수는 있다.
- producer-consumer-problem.md의 깡통-실 프로토콜을 써서, Bob이 문장을 만들면 Alice가 그것을 소비하는 방식으로 처리할 수도 있다.

## 그런데 왜 굳이 새로운 문제로 다루는가

두 방법 모두 대기(waiting)를 요구한다는 한계가 있다. 어느 한쪽이 예기치 못한 지연을 겪으면 상대방도 지연된다. 공유 멀티프로세서 메모리라는 맥락에서 보면, readers-writers 문제는 결국 "여러 메모리 위치의 스냅샷을 한 스레드가 순간적으로 포착하는 방법"의 문제다.

이런 스냅샷을 **대기 없이(without waiting)** 포착할 수 있다면, 즉 읽는 동안 다른 스레드가 그 값을 바꾸는 것을 막지 않고도 일관된 상태를 읽을 수 있다면, 이는 백업이나 디버깅 등 다양한 상황에서 강력한 도구가 된다. 놀랍게도 readers-writers 문제는 대기를 요구하지 않는 해법이 실제로 존재한다.

## Mutual Exclusion / Producer-Consumer와의 관계

세 문제(mutual-exclusion-problem.md, producer-consumer-problem.md, readers-writers.md)는 모두 "여러 스레드가 공유 상태를 안전하게 주고받는" 동일한 뿌리에서 나온 변형이다. 차이는 무엇을 보장해야 하는가에 있다.

- mutual exclusion: 동시에 접근하지 못하게 막는 것 자체가 목표.
- producer-consumer: 생산과 소비의 순서를 맞추는 것이 목표.
- readers-writers: 일관된 상태의 스냅샷을 읽는 것이 목표이며, 유일하게 대기 없는 해법이 가능하다는 점에서 특별하다.

---

참고: producer-consumer-problem.md, parallelization-limits.md
