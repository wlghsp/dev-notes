# Acquire/Release Fence는 생각과 다르게 동작한다

원문: "Acquire and Release Fences Don't Work the Way You'd Expect" — Preshing on Programming (2013)

## Why

acquire-and-release-semantics.md에서 다룬 acquire/release 시맨틱의 정의("release는 앞선 메모리 연산이 그 뒤로 재배치되는 것을 막는다")를, 독립된 fence 명령어(standalone fence)에도 그대로 적용할 수 있다고 생각하기 쉽다. 하지만 C++11에서는 이 정의가 fence에는 적용되지 않는다.

## 왜 이런 오해가 생기는가

C++11 표준은 "store만이 release 연산이 될 수 있고, load만이 acquire 연산이 될 수 있다"고 명시한다. fence는 load도 store도 아니기 때문에, 애초에 release나 acquire 연산으로 분류될 수 없다. 그런데도 사람들은 독립된 release fence가 atomic 변수에 대한 release 연산과 똑같이 동작할 거라고 기대한다.

## 핵심적인 차이

double-checked locking 예시로 비교해보자.

atomic 변수에 직접 release 연산을 쓰는 경우:

```cpp
m_instance.store(tmp, std::memory_order_release);
```

release fence 뒤에 relaxed store를 쓰는 경우:

```cpp
std::atomic_thread_fence(std::memory_order_release);
m_instance.store(tmp, std::memory_order_relaxed);
```

release 연산은 앞선 연산들이 **자기 자신**을 넘어 재배치되는 것만 막는다. 반면 release fence는 앞선 연산들이 **그 뒤에 오는 모든 쓰기**를 넘어 재배치되는 것까지 막아야 한다. 그래서 "release 연산이 release fence를 대신할 수는 없다."

## 실제로 생기는 문제

fence를 별도 atomic 변수의 release 연산으로 바꿔치기하면, 원래 fence가 막아주던 위험이 되살아난다. `m_instance`에 대한 store가 다른 변수(`g_dummy`)에 대한 store보다 앞으로 재배치될 수 있게 되어, 동기화 보장이 깨질 수 있다.

---

참고: acquire-and-release-semantics.md
