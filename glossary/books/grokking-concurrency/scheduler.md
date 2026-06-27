# scheduler

OS의 구성 요소로, 어떤 작업이 언제 CPU를 사용할지 결정한다.

실행 대기 중인 작업들은 ready queue에 줄을 선다. 스케줄러는 이 큐에서 다음 실행할 작업을 선택한다. 선택 기준(정책)에 따라 시스템 전체의 성능 특성이 달라진다.

## 스케줄러의 목표

스케줄러가 동시에 추구하는 목표는 여러 가지이며, 이 목표들은 서로 충돌하는 경우가 많다.

1. throughput — 단위 시간당 완료되는 작업 수를 최대화한다
2. fairness — 각 작업이 CPU 시간을 공평하게 받도록 한다
3. response time — 작업 시작부터 첫 응답까지의 시간을 최소화한다
4. delay — 작업 제출부터 완료까지의 전체 시간을 최소화한다

## time-sharing

여러 작업에 CPU 시간을 나눠주는 방식. 각 작업에 time slice를 할당하고, 시간이 다 되면 다음 작업으로 전환한다.

스케줄러는 예측 불가능하다. 작업의 실행 순서나 각 작업이 얼마나 오래 실행될지를 프로그래머가 가정하고 코드를 짜면 안 된다. 두 작업 중 어느 것이 먼저 끝날지 스케줄러만 알고 있다.

## ready queue

실행 가능한 상태이지만 CPU를 아직 할당받지 못한 작업들이 대기하는 자료구조.

작업의 상태 전환:
- Running → Ready: 타임 슬라이스 소진 후 선점됨
- Blocked → Ready: I/O 완료, 대기 중인 이벤트 발생
- Created → Ready: 새 작업 생성

## CPU-bound vs I/O-bound 구분

스케줄러가 최적으로 동작하려면 각 작업이 CPU-bound인지 I/O-bound인지를 파악해야 한다. I/O-bound 작업이 I/O를 기다리는 동안 CPU-bound 작업이 CPU를 차지하도록 배치하면 CPU가 놀지 않는다.

참고: multitasking.md, cpu-bound.md, io-bound.md, process.md
