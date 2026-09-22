# CountDownLatch 3개로 동시 요청 재현하기

`EnrollmentConcurrencyTest.concurrentEnrollCanExceedCapacityBeforeFix()`에서 쓴 `ready`, `start`, `done` 세 개의 `CountDownLatch`가 각각 무엇을 하는지 정리한다.

---

## 문제: 그냥 스레드를 여러 개 실행하면 안 되는 이유

```java
for (Long memberId : memberIds) {
    executor.submit(() -> enroll(study.getId(), memberId));
}
```

이렇게만 하면 `executor.submit()`이 호출되는 순간 각 스레드는 스레드풀 스케줄링이 되는 대로 제각각 실행된다. 스레드 1이 요청을 다 끝내고 나서야 스레드 2가 시작될 수도 있고, 순서가 흩어질 수도 있다. 그러면 "동시에 30명이 몰렸을 때" 상황이 재현되지 않고, 순차 요청 30번과 비슷해질 위험이 있다. 경합(race condition)은 여러 스레드가 정말 같은 타이밍에 같은 데이터를 건드려야 드러나는 현상이라, 타이밍을 강제로 맞춰줘야 한다.

## CountDownLatch란

숫자 카운터 하나를 들고 있는 객체다. `await()`를 부르면 그 카운터가 0이 될 때까지 스레드가 그 자리에서 블로킹(대기)된다. 다른 스레드가 `countDown()`을 부르면 카운터가 1 줄고, 0이 되는 순간 대기 중이던 모든 스레드가 동시에 풀려난다.

이 테스트에는 latch가 3개 쓰이는데, 역할이 다르다.

```java
CountDownLatch ready = new CountDownLatch(THREAD_COUNT);  // "출발선에 다 모였나?" 확인용
CountDownLatch start = new CountDownLatch(1);             // "출발!" 신호용
CountDownLatch done = new CountDownLatch(THREAD_COUNT);   // "다 끝났나?" 확인용
```

## 흐름을 순서대로 따라가기

```java
for (Long memberId : memberIds) {
    executor.submit(() -> {
        ready.countDown();        // ① "나 준비 끝났어" 보고
        try {
            start.await();        // ② "출발" 신호 올 때까지 여기서 대기
            if (enroll(...).is2xxSuccessful()) {
                successCount.incrementAndGet();
            }
        } finally {
            done.countDown();     // ④ "나 다 끝났어" 보고
        }
    });
}

ready.await();       // ①이 30번 다 될 때까지 메인 스레드가 대기 (=30개 스레드 전부 출발선 도착)
start.countDown();   // ③ 카운트를 1→0으로 만들어서 대기 중인 30개 스레드를 한번에 풀어줌
done.await(30, TimeUnit.SECONDS);  // ④가 30번 다 될 때까지(또는 30초 타임아웃) 메인 스레드가 대기
executor.shutdown();
```

달리기 시합에 비유하면:

- `ready`: 선수 30명이 전부 출발선에 도착했는지 심판이 확인하는 절차. 아직 신호탄은 안 쐈고, 늦게 도착한 선수를 기다려주는 역할.
- `start`: 심판이 실제로 "탕!" 하고 신호탄을 쏘는 순간. 대기하던 30명이 이 순간 동시에 출발.
- `done`: 30명이 다 결승선을 통과했는지(요청을 다 마쳤는지) 심판이 확인.

## 왜 `ready`가 없으면 안 되는가

스레드 1은 `submit` 되자마자 바로 `start.await()`에 진입해 대기하는데, 스레드 30은 스레드풀 스케줄링이 늦어서 아직 `submit`도 안 됐을 수 있다. 그 상태에서 메인 스레드가 곧바로 `start.countDown()`을 불러버리면, 스레드 1~10은 이미 출발했는데 스레드 11~30은 아직 코드 실행도 안 된 상태라 "동시 출발"이 깨진다. `ready.await()`로 "30개 스레드가 전부 준비 완료 지점(`start.await()` 직전)까지 도달했다"는 걸 확인한 뒤에야 신호탄을 쏘는 것이 핵심이다.

## 왜 `done`이 없으면 안 되는가

`done.await()` 없이 바로 `Study reloaded = studyRepository.findById(...)`를 실행하면, 30개 스레드의 요청이 아직 끝나지도 않은 시점에 메인 스레드가 결과를 읽어버린다. 그러면 매번 다른(그리고 대부분 틀린) 값이 찍힌다. `done`은 "30개 요청이 전부 끝날 때까지 결과 확인을 미뤄두는" 역할이다.

## 왜 `ExecutorService`(스레드풀)를 쓰는가

`new Thread(...)`를 30번 직접 만들 수도 있지만, `ExecutorService`(`Executors.newFixedThreadPool(30)`)를 쓰면 스레드 생성·관리를 알아서 해주고, `submit()`한 작업(람다)들을 스레드풀 안의 실제 스레드에 배정해서 병렬 실행해준다. 여기서는 `THREAD_COUNT`(30)만큼 풀을 만들어서 30개 작업이 각자 자기 스레드를 하나씩 배정받아 정말 동시에 돌 수 있게 보장한 것이다. 풀 크기가 스레드 수보다 작으면 일부는 순서를 기다려야 해서 "동시성"이 깨진다.

## 요약

- `ready`: 모든 스레드가 출발 준비를 마칠 때까지 기다림 (스레드풀 스케줄링 지연 보정)
- `start`: 준비가 다 되면 한 번에 신호를 줘서 진짜 동시에 요청을 쏘게 함
- `done`: 모든 요청이 끝날 때까지 메인 테스트 스레드가 기다림 (안 기다리면 결과 검증이 요청 완료 전에 실행돼버림)

이 세 가지가 합쳐져야 "N개의 요청이 정확히 같은 순간에 DB에 부딪힌다"는 상황을 신뢰성 있게 만들 수 있다.
