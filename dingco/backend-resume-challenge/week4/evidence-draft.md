# Week 4 증거 초안

아래 내용을 대상 저장소의 `evidence/week-04__weekly-pr.md`에 옮긴다.

```markdown
# Week 4 증거

## 변경

- 대상 API: studypass의 `POST /api/enrollments/studies/{studyId}`
- 변경한 이력서 문장: `resume/resume.md` 문장 4
- 개인정보·회사 기밀 제거 확인: 해당 없음
- 선택한 락 전략: 비관적 락. `EnrollmentService.enroll()`에 `findByIdForUpdate`(`@Lock(PESSIMISTIC_WRITE)`)를 적용했다. 낙관적 락(`@Version` + 재시도)은 비교용으로만 `/optimistic` 엔드포인트에 남겼다.
- 부수 수정: `MemberEnrollmentStatsRepository.increment()`가 테스트 환경(H2)에서 깨지던 MySQL 전용 UPSERT 문법을 UPDATE-then-INSERT로 고쳤다.
- 회귀 테스트: `EnrollmentConcurrencyTest.java` — 단일 스레드, 개선 전/후, 낙관적 락 비교 4개.

## 검증

- 환경: 로컬 macOS / Java 17 / Spring Boot 3.5 / MySQL 8.0(도커) / 테스트는 H2(`MODE=MySQL`)
- 고정 조건: capacity=10, 동시 요청 30건, `CountDownLatch` 3개(`ready`/`start`/`done`)로 동시 출발
- 판정 기준: 성공 응답 수(`successCount`), 실제 저장 건수(`countByStudyId`), 최종 `enrolledCount`, 재시도 총 횟수(`EnrollmentOptimisticFacade.getRetryCount()`)

### 개선 전 — 락 미적용

```text
[개선 전] successCount=30, savedEnrollmentCount=30, enrolledCount=4, capacity=10
```

30건 신청이 전부 성공·저장됐지만(`successCount == savedEnrollmentCount == 30`) `Study.enrolledCount`는 4에 그쳤다. 정원 "초과"가 아니라 "미달" 형태의 Lost Update — 여러 트랜잭션이 같은 스냅샷을 기준으로 `+1`을 계산해 서로의 갱신을 덮어썼다.

(주의: 락 교체 후 같은 테스트를 재실행하면 `enrolledCount`가 깨지지 않는다 — 이는 재현 실패가 아니라 테스트 대상 코드 자체가 이미 락 버전으로 바뀌었기 때문이다. 개선 전 근거는 반드시 코드 교체 **이전** 실행 값만 쓴다.)

### 개선 후 — 비관적 락 (운영 경로, `@RepeatedTest(5)`)

```text
repetition 1 of 5 PASSED (0.653s)
repetition 2 of 5 PASSED (0.151s)
repetition 3 of 5 PASSED (0.118s)
repetition 4 of 5 PASSED (0.115s)
repetition 5 of 5 PASSED (0.1s)
```

5회 모두 `enrolledCount == successCount == capacity(10)`, 실패 응답 0건. 재시도 수: 없음 — 락 대기로 처리.

### 낙관적 락 비교 (`/optimistic`, `@RepeatedTest(5)`)

```text
successCount=10, failureCount=20, enrolledCount=10, capacity=10, retryCount=47
successCount=10, failureCount=20, enrolledCount=10, capacity=10, retryCount=59
successCount=10, failureCount=20, enrolledCount=10, capacity=10, retryCount=56
successCount=10, failureCount=20, enrolledCount=10, capacity=10, retryCount=37
successCount=10, failureCount=20, enrolledCount=10, capacity=10, retryCount=33
```

정합성(`enrolledCount == capacity`)은 지켜졌지만, 재시도 5회(MAX_RETRY)를 소진한 20건은 매회 실패 응답을 받았다. 재시도 총 횟수는 33~59회로 실행마다 편차가 컸다(스레드 스케줄링에 따른 충돌 빈도 차이).

### 단일 스레드 테스트

```text
sequentialEnrollDoesNotExceedCapacity PASSED (0.072s)
```

### 전체 테스트 실행

```text
BUILD SUCCESSFUL
EnrollmentConcurrencyTest 전체 케이스 통과, 실패 0건
```

## 선택 근거

같은 조건(capacity=10, 동시 요청 30건)에서 두 전략을 비교했다.

| 항목 | 비관적 락 | 낙관적 락(MAX_RETRY=5) |
| --- | --- | --- |
| 정합성 | 5회 모두 유지 | 5회 모두 유지 |
| 성공 수 | 5회 모두 10/10, 실패 0건 | 5회 모두 10/10, 매회 20건 실패 |
| 재시도 수 | 없음(대기로 처리) | 33~59회(편차 큼) |

정합성은 동률이지만 성공 수·재시도 수에서 비관적 락이 우위라 비관적 락을 최종 채택했다. 인기 스터디에 순간적으로 요청이 몰리는 패턴과 짧은 트랜잭션이라는 이 프로젝트 상황과도 맞는다.

버린 대안(낙관적 락)이 더 적합해지는 조건: 충돌이 지금보다 훨씬 드물어 재시도 비용이 낮아지는 경우.

## 트랜잭션 범위 점검

- `EnrollmentService.enroll()` 안에 트랜잭션 밖으로 뺄 외부 호출(이메일 발송 등)은 없었다.
- 회원 조회(`memberRepository.findById`)를 락 획득(`findByIdForUpdate`) 이전으로 배치해, 정원 경합과 무관한 조회가 락 보유 시간에 포함되지 않도록 했다.

## 근거형 질문

### 질문 1: 동시성 문제가 발생할 가능성이 있는 시나리오는?

- 최초 판단: 인기 스터디 정원 마감 직전 동시 신청.
- 검증 후 답변: 재현됨. capacity=10에 동시 요청 30건 → `successCount=savedEnrollmentCount=30`인데 `enrolledCount=4`(미달 형태 Lost Update).

### 질문 2: 어떤 동시성 제어 방법을 고려했는가?

- 최초 판단: REPEATABLE READ만으로는 Lost Update를 막지 못하므로 비관적/낙관적 락 중 하나 필요.
- 검증 후 답변: 둘 다 구현해 비교. 정합성은 동률, 성공 수·재시도 수는 비관적 락이 우위(실패 0건·재시도 없음 vs 매회 20건 실패·재시도 33~59회)라 비관적 락을 채택.

### 질문 3: 트랜잭션 범위를 어떻게 설정했는가?

- 최초 판단: 외부 호출 없음, 락과 무관한 조회를 락 획득 전후로 재배치.
- 검증 후 답변: 외부 호출은 없었고, 회원 조회를 락 획득 이전으로 옮겨 락 보유 시간을 줄였다.

### 질문 4: 개선 전후를 어떻게 측정·검증했는가?

- 최초 판단: `ExecutorService` + `CountDownLatch`로 동일 조건 동시 요청을 재현하고 전/후를 비교.
- 검증 후 답변: 같은 capacity·스레드 수로 개선 전(1회, 붕괴 확인) → 개선 후(5회 반복, 전부 통과) 순서로 측정해 재현율까지 남겼다.

## 리뷰 반영

- 최초 제출: 자동 리뷰 수신 전
- 재제출: [리뷰 수신 후 지적 내용과 보완 파일을 기록]
```
