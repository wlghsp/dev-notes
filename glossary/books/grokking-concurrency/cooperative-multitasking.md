# cooperative multitasking

협력형 멀티태스킹. 작업이 스스로 CPU 제어권을 양보해야만 다음 작업으로 전환이 일어나는 방식.

OS 스케줄러가 강제로 개입하지 않는다. 각 작업이 "나 이제 양보할게"라고 명시적으로 신호를 보내야 전환이 된다.

## preemptive multitasking과의 차이

preemptive(선점형) — OS가 타이머 인터럽트로 실행 중인 작업을 강제로 중단시킨다. 작업의 의지와 무관하게 전환된다.

cooperative(협력형) — 작업 스스로 양보 시점을 결정한다. OS는 기다릴 뿐이다.

## 단점

작업이 양보하지 않으면 다른 작업이 영원히 실행되지 않는다. 무한 루프나 버그가 있는 작업 하나가 전체 시스템을 멈출 수 있다.

이 취약점 때문에 현대 범용 OS(Linux, Windows, macOS)는 선점형 방식을 사용한다.

## 어디서 쓰이나

현대 OS 수준에서는 거의 사라졌지만, 애플리케이션 레벨에서는 여전히 쓰인다.

Python의 asyncio, JavaScript의 이벤트 루프가 대표적이다. async/await로 작성된 코드에서 `await` 키워드가 바로 협력형 양보 지점이다. 작업이 `await`를 만나면 스스로 제어권을 이벤트 루프에 돌려준다.

참고: multitasking.md, scheduler.md
