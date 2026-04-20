# 이벤트루프는 언어마다 어떻게 다른가

> 핵심 메커니즘은 같다 — OS의 I/O 멀티플렉싱. 차이는 "완료된 I/O를 어떻게 처리하느냐"다.

---

## 공통 기반: OS의 I/O 멀티플렉싱

어떤 런타임이든 결국 OS에 이렇게 요청한다:

```
"이 소켓들 감시해줘. 뭐라도 완료되면 알려줘."
    → Linux:   epoll
    → macOS:   kqueue
    → Windows: IOCP
```

OS가 수천 개의 소켓을 동시에 감시하다가 응답이 오면 런타임에 알린다. 스레드가 블로킹되지 않는다. 이 부분은 모든 런타임이 동일하다.

차이는 그 이후다 — **완료 신호를 받은 뒤 어떻게 코드를 실행하는가.**

---

## Node.js — 콜백 큐 + 싱글 JS 스레드

```
OS 완료 신호 → 콜백 큐에 적재 → 이벤트루프가 꺼내서 실행
```

JS 코드를 실행하는 스레드는 단 하나다. 콜백이 큐에 쌓이면 순서대로 실행한다.

```javascript
// await마다 이벤트루프에 제어권을 돌려줌
const user = await db.findById(id);     // OS에 위임, 돌아옴
const orders = await api.getOrders(id); // OS에 위임, 돌아옴
res.json({ user, orders });
```

- **장점**: 구조 단순, 동시성 버그 없음 (공유 상태가 없음)
- **단점**: CPU 집약 작업이 이벤트루프를 막음. 멀티코어 활용 불가 (기본)

---

## Python asyncio — 코루틴 + 싱글 스레드

Node.js와 구조가 거의 동일하다. 차이는 콜백 대신 **코루틴**을 사용한다는 것.

```python
# await마다 이벤트루프에 제어권 반환
async def handle(request):
    user = await db.find_by_id(id)      # 이벤트루프로 돌아감
    orders = await api.get_orders(id)   # 이벤트루프로 돌아감
    return Response(user, orders)
```

코루틴은 `await` 지점에서 실행을 멈추고 이벤트루프에 제어권을 넘긴다. 이벤트루프는 다른 코루틴을 실행하다가 I/O가 완료되면 돌아온다.

```
이벤트루프:
  코루틴A 실행 → await → 일시정지
  코루틴B 실행 → await → 일시정지
  코루틴C 실행 → 완료
  OS: A의 I/O 완료 → 코루틴A 재개
```

- **장점**: Node.js보다 명시적인 제어 흐름, 기존 동기 코드와 혼용 가능
- **단점**: `async/await`가 전파된다 — 한 곳이 async면 호출 스택 전체가 async여야 함. Node.js와 동일한 멀티코어 한계.

---

## Go — 고루틴 + M:N 스케줄링

Go는 다르다. 이벤트루프가 언어 런타임 안에 내장되어 있고, **멀티코어를 기본으로 활용**한다.

```go
// 겉보기엔 동기 코드처럼 보임
func handle(w http.ResponseWriter, r *http.Request) {
    user := db.FindById(id)      // 여기서 고루틴이 멈추지만 OS 스레드는 안 멈춤
    orders := api.GetOrders(id)  // 동일
    json.NewEncoder(w).Encode(Response{user, orders})
}
```

Go 런타임이 내부적으로 처리하는 것:

```
고루틴 (수천~수만 개, 경량)
    ↓ M:N 스케줄링
OS 스레드 (GOMAXPROCS 개수, 보통 CPU 코어 수)
    ↓
epoll/kqueue (OS I/O 멀티플렉싱)
```

고루틴이 I/O로 블로킹되면 Go 런타임이 해당 OS 스레드에서 다른 고루틴을 실행한다. OS 스레드는 놀지 않는다.

- **장점**: 비동기 코드처럼 보이지 않음 (동기 스타일 유지). 멀티코어 자동 활용. 고루틴은 스택 2KB로 시작 (스레드의 1/500).
- **단점**: 런타임 복잡도가 높음. 데이터 경쟁(Race Condition) 가능성 존재.

---

## Java Netty / Spring WebFlux — 이벤트루프 스레드 풀

Java는 전통적으로 스레드 기반(Spring MVC)이었지만, Netty 위에서 동작하는 WebFlux는 이벤트루프 방식을 택했다.

```
이벤트루프 스레드: CPU 코어 수만큼 (보통 4~8개)
    각 스레드가 독립적인 이벤트루프를 돌림
    → Nginx의 워커 프로세스와 유사한 구조
```

```java
// WebFlux — 리액티브 스트림
Mono<Response> handle(ServerRequest req) {
    return userRepo.findById(id)           // 비동기
        .zipWith(api.getOrders(id))        // 병렬 비동기
        .map(tuple -> new Response(tuple.getT1(), tuple.getT2()));
}
```

- **장점**: 멀티코어 활용 + 이벤트루프의 낮은 메모리 사용
- **단점**: 리액티브 스트림 패러다임이 학습 곡선 높음. 기존 Spring MVC 코드와 혼용 어려움.

> Java 21+의 Virtual Thread(가상 스레드)는 Go의 고루틴과 유사한 방식으로 이 문제를 다시 접근한다. 동기 스타일로 작성해도 내부적으로 논블로킹으로 동작한다.

---

## Nginx — 워커 프로세스당 이벤트루프

Nginx는 서버 프로세스 자체가 이벤트루프 구조다.

```
마스터 프로세스
    ├── 워커 프로세스 1 (이벤트루프, CPU 코어 0)
    ├── 워커 프로세스 2 (이벤트루프, CPU 코어 1)
    ├── 워커 프로세스 3 (이벤트루프, CPU 코어 2)
    └── 워커 프로세스 4 (이벤트루프, CPU 코어 3)
```

각 워커 프로세스가 독립적인 이벤트루프를 돌리며 수천 개의 연결을 처리한다. 스레드가 아닌 프로세스 단위라 메모리를 공유하지 않아 안전하다.

---

## 비교 한눈에

| | Node.js | Python asyncio | Go | Java WebFlux | Nginx |
|---|---|---|---|---|---|
| 실행 스레드 | 1개 (JS) | 1개 | 코어 수 | 코어 수 | 코어 수 (프로세스) |
| 코드 스타일 | async/await | async/await | 동기처럼 보임 | 리액티브 스트림 | C (설정 기반) |
| 멀티코어 활용 | 기본 불가 | 기본 불가 | 자동 | 가능 | 가능 |
| I/O 처리 | OS epoll + libuv | OS epoll | OS epoll + 런타임 | Netty + OS epoll | OS epoll |
| 학습 곡선 | 낮음 | 낮음 | 중간 | 높음 | 낮음 (설정) |

---

## 결론

> 어떤 런타임이든 OS의 I/O 멀티플렉싱(`epoll`/`kqueue`)을 쓴다는 건 동일하다.
>
> 차이는 **"완료 신호를 받은 뒤 코드를 어떻게 실행하는가"** — 콜백, 코루틴, 고루틴, 리액티브 스트림. 각자 다른 방식으로 같은 문제를 푼다.
>
> Go가 주목받는 이유는 동기 스타일 코드로 멀티코어 이벤트루프를 자동으로 활용하기 때문이다. 비동기의 이점을 개발자가 신경 쓰지 않아도 런타임이 처리한다.
