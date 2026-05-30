# gc-overview
참고: Java Performance: The Definitive Guide Ch.5

---

GC(Garbage Collection)가 살아있는 객체와 죽은 객체를 어떻게 구분하고, 죽은 객체를 어떻게 회수하는가.

왜 필요한지는 gc-why.md 참고.

---

## 기본 동작: Mark → Sweep → Compact

GC는 크게 세 단계로 동작한다.

1. Mark — GC Root(스택의 로컬 변수, static 필드, JNI 참조)에서 시작해 참조를 따라가며 도달 가능한 모든 객체에 "살아있음" 표시를 한다. 표시되지 않은 객체는 죽은 객체다.

2. Sweep — 죽은 객체가 차지하던 메모리를 빈 공간으로 표시한다. 아직 실제로 지우지는 않는다.

3. Compact — 살아있는 객체들을 한쪽으로 모아 단편화(fragmentation)를 없앤다. 새 객체 할당이 연속된 빈 공간에 빠르게 이뤄지도록 한다.

모든 GC 알고리즘이 세 단계를 다 하는 건 아니다. Compact를 생략하는 알고리즘도 있고(CMS), Mark와 Sweep을 동시에 하는 것도 있다.

---

## GC Root란

GC가 살아있는 객체를 추적하기 시작하는 출발점이다. GC Root에서 참조를 따라가다 만날 수 있으면 살아있는 객체, 만날 수 없으면 죽은 객체다.

GC Root의 종류:
- 각 스레드의 스택에 있는 로컬 변수와 파라미터
- 클래스의 static 필드
- JNI 참조(C/C++ 코드에서 참조하는 Java 객체)
- 활성화된 스레드 자체

---

## Stop-The-World

Mark 단계에서 힙을 일관성 있게 탐색하려면, 탐색 도중에 객체 참조가 바뀌면 안 된다. 그래서 GC의 일부 또는 전체 단계에서 애플리케이션 스레드를 모두 멈춘다. 이것이 Stop-The-World(STW)다.

STW가 길수록 애플리케이션의 latency가 나빠진다. GC 알고리즘의 발전 방향은 대부분 STW를 줄이는 쪽이다. 자세한 내용은 stop-the-world.md 참고.

---

## Generational GC

현대 JVM의 GC는 대부분 세대(generation)를 나눈다. 객체 대부분은 금방 죽는다는 경험적 관찰(weak generational hypothesis)에 기반한다.

- Young Generation: 새로 생성된 객체가 들어온다. GC가 자주 실행되고(minor GC), 대부분 여기서 회수된다.
- Old Generation: Young에서 살아남은 객체가 이동한다. GC가 드물게 실행되지만(major GC), 실행되면 더 오래 걸린다.

세대를 나누면 매번 전체 힙을 탐색하지 않아도 되므로 GC 효율이 높아진다. 자세한 내용은 generational-gc.md 참고.
