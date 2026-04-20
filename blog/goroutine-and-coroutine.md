# 고루틴과 코루틴 — 같은 뿌리, 다른 방향

> "이름이 비슷한데 관련 있는 건가?" 있다. 고루틴은 코루틴 개념에서 출발했다. 다만 Go가 그것을 어떻게 구현했느냐가 다르다.

---

## 코루틴 — 언어를 가로지르는 보편 개념

코루틴(Coroutine)은 특정 언어의 기능이 아니라 **프로그래밍 개념**이다.

1958년 Melvin Conway가 어셈블리 작성 중 처음 정의했고,  
이후 Python, JavaScript, Kotlin, C#, Lua, Ruby, Go 등 수많은 언어가 각자의 방식으로 구현했다.

핵심 아이디어는 하나다:

> **실행을 중간에 멈추고, 나중에 그 지점부터 재개할 수 있는 함수**

```
일반 함수:  호출 → [----실행----] → 반환

코루틴:     호출 → [--실행--] → 중단 → [--재개--] → 중단 → [--완료--]
                              ↑ 여기서 제어권을 넘길 수 있다
```

### 각 언어의 코루틴 구현

코루틴이 범용 개념이다보니, 언어마다 문법은 달라도 동작 원리는 같다.

**Python — generator / async**

```python
# generator 방식 (yield)
def counter():
    i = 0
    while True:
        yield i      # 여기서 중단, 값 반환
        i += 1

gen = counter()
print(next(gen))  # 0
print(next(gen))  # 1

# async/await 방식
async def fetch_data():
    await asyncio.sleep(1)   # 여기서 중단, 이벤트루프에 제어권 반환
    return "data"
```

**JavaScript — async/await**

```javascript
async function fetchUser(id) {
    const user = await fetch(`/api/users/${id}`);  // 중단 지점
    return user.json();
}
```

**Kotlin — suspend fun**

```kotlin
suspend fun fetchUser(id: Long): User {
    delay(1000)           // 중단 지점 (스레드 블로킹 없이 대기)
    return userRepository.findById(id)
}
```

**C# — async/await**

```csharp
async Task<User> FetchUserAsync(long id) {
    await Task.Delay(1000);   // 중단 지점
    return await repository.FindByIdAsync(id);
}
```

문법은 다르지만 모두 같은 개념이다: **실행을 중단하고, 나중에 재개한다.**

### 코루틴의 특성 — 협력적 스케줄링

코루틴은 기본적으로 **협력적(cooperative)** 이다.  
코루틴 자신이 `yield` / `await` / `suspend`를 호출해야 제어권이 넘어간다.

```
코루틴 A 실행 중
    → A가 yield 호출 → 제어권 이동
        → 코루틴 B 실행 중
            → B가 yield 호출 → 제어권 이동
                → 코루틴 A 재개
```

A가 yield를 호출하지 않으면 B는 실행 기회를 얻지 못한다.  
런타임이 강제로 끼어들지 않는다.

그리고 코루틴은 대부분 **싱글스레드** 위에서 동작한다.  
실제로 동시에 실행되는 게 아니라, 빠르게 번갈아 실행되는 것처럼 보이는 것이다.

---

## 고루틴 — Go가 코루틴을 재해석한 것

고루틴(Goroutine)은 이름에서 알 수 있듯 코루틴에서 출발했다.  
그러나 Go가 목표로 한 것은 다른 언어의 코루틴과 방향이 달랐다.

> "코루틴의 장점(가볍고, 많이 만들 수 있고)을 유지하면서,  
> 진짜 병렬 실행까지 가능하게 만들자."

결과적으로 고루틴은 코루틴과 세 가지가 달라졌다.

### 1. 선점적 스케줄링

코루틴은 스스로 yield해야 제어권이 넘어간다.  
고루틴은 **Go 런타임이 강제로 전환**할 수 있다. (Go 1.14부터 완전한 선점적 스케줄링)

```go
// yield 같은 게 없다. 그냥 실행하면 런타임이 알아서 스케줄한다.
go func() {
    for {
        // 무한루프여도 런타임이 개입해서 다른 고루틴에 CPU를 넘긴다
        doWork()
    }
}()
```

코루틴 방식이었다면 이 무한루프는 다른 코루틴을 굶길 것이다.

### 2. 진짜 병렬 실행

코루틴은 싱글스레드 위에서 번갈아 실행된다.  
고루틴은 **여러 OS 스레드에 분산**되어 멀티코어를 활용한다.

```
고루틴 수백만 개
      ↓  (Go 런타임 스케줄러 — M:N 멀티플렉싱)
OS 스레드 수십 개 (GOMAXPROCS 기본값 = CPU 코어 수)
      ↓
CPU 코어들 (병렬 실행)
```

코루틴이 "동시성(concurrency)"이라면, 고루틴은 "동시성 + 병렬성(parallelism)" 둘 다 된다.

### 3. yield 없이 `go` 하나로

코루틴은 어디서 중단할지 개발자가 명시한다. (`yield`, `await`, `suspend`)  
고루틴은 **중단 지점을 선언하지 않는다.** 그냥 `go`로 실행하면 런타임이 처리한다.

```go
func sayHello(name string) {
    fmt.Println("hello,", name)
}

go sayHello("world")   // 이게 전부. suspend도 yield도 없다.
```

개발자가 제어 흐름을 직접 설계할 필요가 없다.

---

## 비교 정리

| 항목 | 코루틴 | 고루틴 |
|------|--------|--------|
| 개념 범위 | 언어 전반에 사용되는 범용 개념 | Go 언어의 구체적 구현 |
| 스케줄링 | 협력적 (개발자가 yield) | 선점적 (런타임이 자동 전환) |
| 병렬 실행 | 기본적으로 불가 (싱글스레드) | 가능 (멀티코어 활용) |
| 중단 선언 | 명시적 (yield/await/suspend) | 없음 (런타임에 위임) |
| 메모리 | 언어마다 다름 | 초기 2KB, 동적 확장 |
| 관리 주체 | 이벤트루프 or 코루틴 스케줄러 | Go 런타임 스케줄러 |

### 스레드와의 비교

고루틴이 코루틴과 달라졌다고 해서 OS 스레드와 같은 건 아니다.

| 항목 | OS 스레드 | 고루틴 |
|------|-----------|--------|
| 초기 스택 크기 | 1~8MB | 2KB |
| 관리 주체 | OS 커널 | Go 런타임 |
| 생성 비용 | 높음 | 매우 낮음 |
| Context Switch | 커널 모드 전환 | 유저 모드 전환 |
| 동시 실행 개수 | 수천 | 수백만 |

고루틴은 OS 스레드보다 훨씬 가볍고, 코루틴보다 진짜 병렬성에 가깝다.  
그 중간 어딘가에 있다.

---

## 고루틴의 통신 — 채널

코루틴은 보통 반환값이나 공유 상태로 데이터를 주고받는다.  
고루틴은 **채널(channel)** 을 통한 메시지 패싱을 권장한다.

```go
func produce(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}

func main() {
    ch := make(chan int)
    go produce(ch)

    for v := range ch {
        fmt.Println(v)  // 0, 1, 2, 3, 4
    }
}
```

> "메모리를 공유해서 통신하지 말고, 통신해서 메모리를 공유하라"  
> — Go 설계 철학

락(lock) 없이 고루틴 간 데이터를 안전하게 전달한다.

---

## 정리

코루틴은 1958년부터 이어져온 **범용 개념**이다.  
Python의 `yield`, JavaScript의 `async/await`, Kotlin의 `suspend fun` — 전부 코루틴이다.

고루틴은 Go가 그 개념을 가져와 **런타임 스케줄러 + 멀티코어 병렬성**을 더한 것이다.  
이름이 비슷한 건 우연이 아니라, 실제로 같은 뿌리에서 나온 것이다.

```
코루틴:  범용 개념 — "실행을 멈추고 재개한다"
            └─ Python yield / asyncio
            └─ JavaScript async/await
            └─ Kotlin suspend
            └─ C# async/await
            └─ ...

고루틴:  Go의 구현 — 코루틴 + 선점적 스케줄링 + 멀티코어 병렬성
```

> 코루틴을 알면 고루틴이 왜 그렇게 설계됐는지 이해가 된다.  
> 고루틴은 "코루틴의 불편함(yield 관리, 싱글스레드 한계)을 런타임이 대신 해결한 것"이다.
