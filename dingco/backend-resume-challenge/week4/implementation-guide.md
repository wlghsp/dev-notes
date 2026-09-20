# 4주차 구현 가이드

`week4/prep-questions.md`는 아직 답변이 비어 있다. 이 문서는 그 답을 전제로 하지 않고, `missions/README.md` Week 4 요구사항과 `studypass`(`/Users/jihochoi/Documents/study/dingco/challenge-backend-resume-2026-08-wlghsp-r17`) 현재 코드를 근거로 "무엇을 어떤 순서로 구현할지"만 정리한다. 실행 결과·수치·판단 근거는 실제로 실행한 뒤 evidence와 prep-questions에 채운다.

---

## 0. 현재 코드 상태 확인

경합 지점은 이미 두 곳에 주석으로 표시되어 있다.

- `Study.enroll()` (`Study.java`): `isFull()`로 `enrolledCount >= capacity`를 확인한 뒤 `enrolledCount + 1`. check와 act 사이에 원자성이 없다.
- `EnrollmentService.enroll()` (`EnrollmentService.java`): `@Transactional`이지만 study 조회 → `study.enroll()` → Enrollment 저장 → 집계 갱신 순서로만 되어 있고 락이나 원자적 갱신은 없다.

`EnrollmentController.enroll()`은 `POST /api/enrollments/studies/{studyId}?memberId=`로 이미 노출되어 있으므로 컨트롤러 변경은 필요 없다.

## 1단계: 브랜치 생성

`submit/week-04__weekly-pr` 브랜치 생성 및 체크아웃 완료.

- [x] 브랜치 생성 완료

## 2단계: 정원 초과 재현 테스트 작성 (개선 전)

기존 `EnrollmentControllerTest.java`는 `@SpringBootTest(webEnvironment = RANDOM_PORT)` + `TestRestTemplate`로 실제 HTTP 호출을 검증하고, `@Transactional`(자동 롤백)을 쓰지 않는다. 동시성 재현도 이 스타일을 그대로 따라야 한다 — 각 스레드의 요청이 독립 커넥션·독립 트랜잭션으로 실제 커밋돼야 경합이 재현되기 때문이다.

새 파일 `src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentConcurrencyTest.java`로 분리한다(기존 파일은 3주차 search/stats 테스트 전용으로 두고 섞지 않는다).

```java
package co.dingcodingco.studypass.enrollment;

import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.study.Study;
import co.dingcodingco.studypass.study.StudyRepository;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class EnrollmentConcurrencyTest {

    private static final int CAPACITY = 10;
    private static final int THREAD_COUNT = 30;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private MemberRepository memberRepository;

    @Autowired
    private StudyRepository studyRepository;

    @DisplayName("단일 스레드로 capacity만큼 순차 신청하면 전부 성공하고 정원을 넘기지 않는다")
    @Test
    void sequentialEnrollDoesNotExceedCapacity() {
        Study study = saveStudy();
        List<Long> memberIds = saveMembers(CAPACITY);

        int successCount = 0;
        for (Long memberId : memberIds) {
            if (enroll(study.getId(), memberId).is2xxSuccessful()) {
                successCount++;
            }
        }

        Study reloaded = studyRepository.findById(study.getId()).orElseThrow();
        assertThat(successCount).isEqualTo(CAPACITY);
        assertThat(reloaded.getEnrolledCount()).isEqualTo(CAPACITY);
    }

    @DisplayName("동시 요청 " + THREAD_COUNT + "건이 capacity " + CAPACITY + "명 스터디에 몰리면 정원을 넘겨 신청될 수 있다 (개선 전)")
    @Test
    void concurrentEnrollCanExceedCapacityBeforeFix() throws InterruptedException {
        Study study = saveStudy();
        List<Long> memberIds = saveMembers(THREAD_COUNT);

        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        CountDownLatch ready = new CountDownLatch(THREAD_COUNT);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(THREAD_COUNT);
        AtomicInteger successCount = new AtomicInteger();

        for (Long memberId : memberIds) {
            executor.submit(() -> {
                ready.countDown();
                try {
                    start.await();
                    if (enroll(study.getId(), memberId).is2xxSuccessful()) {
                        successCount.incrementAndGet();
                    }
                } catch (Exception ignored) {
                    // 정원 마감 등으로 실패한 요청은 실패로만 집계한다.
                } finally {
                    done.countDown();
                }
            });
        }

        ready.await();
        start.countDown();
        done.await(30, TimeUnit.SECONDS);
        executor.shutdown();

        Study reloaded = studyRepository.findById(study.getId()).orElseThrow();

        // 개선 전에는 assert로 강제하지 않고, 실제로 넘긴 값을 그대로 기록해 evidence에 옮긴다.
        System.out.printf(
                "[개선 전] successCount=%d, enrolledCount=%d, capacity=%d%n",
                successCount.get(), reloaded.getEnrolledCount(), CAPACITY);
    }

    private ResponseEntity<Void> enroll(Long studyId, Long memberId) {
        return restTemplate.exchange(
                "/api/enrollments/studies/" + studyId + "?memberId=" + memberId,
                HttpMethod.POST,
                null,
                Void.class);
    }

    private Study saveStudy() {
        return studyRepository.save(
                new Study("동시성 테스트 스터디", "BACKEND", 50_000, CAPACITY,
                        LocalDateTime.of(2026, 7, 1, 0, 0)));
    }

    private List<Long> saveMembers(int count) {
        List<Long> ids = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            String suffix = UUID.randomUUID().toString();
            Member member = memberRepository.save(
                    new Member("concurrency-" + suffix + "@studypass.test", "동시성 테스트 회원 " + i));
            ids.add(member.getId());
        }
        return ids;
    }
}
```

`enroll()`이 실패 시 무슨 상태 코드를 내는지(현재 `EnrollmentService.enroll()`은 `IllegalStateException`을 던지므로, 전역 예외 처리기가 없다면 500이 반환된다) 실제 실행 후 확인하고, 필요하면 `is2xxSuccessful()` 대신 정확한 상태 코드로 성공 판정 조건을 좁힌다.

같은 테스트를 5회 반복 실행해(`./gradlew test --tests` 반복 또는 `@RepeatedTest(5)`) "몇 번 중 몇 번 정원 초과가 나타났는지" 재현율을 기록한다. 매번 넘기지 않을 수도 있다는 점 자체가 prep-questions 3번(재현 테스트가 매번 같은 결과가 나오지 않는 이유)의 근거가 된다.

- [ ] 단일 스레드로 capacity만큼 순차 신청 → 통과(정원 안 넘음)를 보여주는 테스트
- [ ] 동시 스레드로 capacity 초과 시도 → 정원 초과가 재현됨을 보여주는 테스트(개선 전)
- [ ] 재현 시도 결과(성공 신청 수, 최종 enrolledCount)를 콘솔 출력 또는 로그로 남김
- [ ] 반복 실행 결과(몇 회 중 몇 회 초과) 기록

## 3단계: 격리 수준만으로 안 막히는 이유 확인

별도 구현은 필요 없다. `@Transactional`의 기본 격리 수준(REPEATABLE READ, MySQL 기본값)에서도 2단계 테스트가 여전히 정원을 넘긴다는 사실 자체가 근거다. 각 트랜잭션이 자신이 읽은 `enrolledCount` 스냅샷을 기준으로 +1해서 커밋하므로, 격리 수준이 동시 트랜잭션 각각의 커밋을 막지는 않는다. 이 설명은 구현 후 prep-questions 2번에 채운다.

## 4단계: 락 전략 선택 및 구현

미션은 낙관적/비관적 중 **하나만** 선택하면 된다. 이 프로젝트 상황(짧은 트랜잭션, 인기 스터디에 순간적으로 요청이 몰림)에서는 비관적 락이 기본 후보다 — 경합이 몰리는 짧은 트랜잭션은 재시도 로직 없이 대기만으로 정합성을 보장하기 쉽다. 최종 선택 근거는 실측(성공 수·재시도 수) 후 prep-questions 4번에 적는다.

### 4-A. 비관적 락으로 구현할 경우

`StudyRepository.java`에 락 조회 메서드를 추가한다. 기존 메서드(`findByCategory`, `findTop10ByStatusOrderByEnrolledCountDesc`, `findByIdGreaterThanOrderByIdAsc`)는 그대로 두고 아래만 더한다.

```java
package co.dingcodingco.studypass.study;

import jakarta.persistence.LockModeType;
import java.util.List;
import java.util.Optional;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface StudyRepository extends JpaRepository<Study, Long> {

    List<Study> findByCategory(String category);

    List<Study> findTop10ByStatusOrderByEnrolledCountDesc(String status);

    List<Study> findByIdGreaterThanOrderByIdAsc(Long cursor, Pageable pageable);

    /**
     * 4주차 미션의 비관적 락 조회다. SELECT ... FOR UPDATE로 행을 잠가
     * 다른 트랜잭션이 이 study의 갱신을 끝날 때까지 대기하게 만든다.
     */
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select s from Study s where s.id = :id")
    Optional<Study> findByIdForUpdate(@Param("id") Long id);
}
```

`EnrollmentService.java`에서 기존 `studyRepository.findById(studyId)`를 `findByIdForUpdate(studyId)`로 교체한다.

```java
package co.dingcodingco.studypass.enrollment;

import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.study.Study;
import co.dingcodingco.studypass.study.StudyRepository;
import java.time.LocalDateTime;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class EnrollmentService {

    private final StudyRepository studyRepository;
    private final MemberRepository memberRepository;
    private final EnrollmentRepository enrollmentRepository;
    private final MemberEnrollmentStatsRepository memberEnrollmentStatsRepository;

    public EnrollmentService(
            StudyRepository studyRepository,
            MemberRepository memberRepository,
            EnrollmentRepository enrollmentRepository,
            MemberEnrollmentStatsRepository memberEnrollmentStatsRepository) {
        this.studyRepository = studyRepository;
        this.memberRepository = memberRepository;
        this.enrollmentRepository = enrollmentRepository;
        this.memberEnrollmentStatsRepository = memberEnrollmentStatsRepository;
    }

    /**
     * 4주차 미션: SELECT ... FOR UPDATE로 study 행을 잠가 정원 경합을 막는다.
     * 회원 조회는 락과 무관하므로 락을 잡기 전에 먼저 끝낸다(5단계 트랜잭션 범위 조정).
     */
    @Transactional
    public Long enroll(Long studyId, Long memberId) {
        Member member = memberRepository.findById(memberId)
                .orElseThrow(() -> new IllegalArgumentException("회원을 찾을 수 없습니다: " + memberId));
        Study study = studyRepository.findByIdForUpdate(studyId)
                .orElseThrow(() -> new IllegalArgumentException("스터디를 찾을 수 없습니다: " + studyId));

        study.enroll();

        LocalDateTime enrolledAt = LocalDateTime.now();
        Enrollment enrollment =
                new Enrollment(study, member, "CONFIRMED", study.getFee(), enrolledAt);
        Enrollment saved = enrollmentRepository.save(enrollment);

        memberEnrollmentStatsRepository.increment(
                member.getId(), study.getFee(), enrolledAt
        );

        return saved.getId();
    }
}
```

락을 건 두 번째 트랜잭션부터는 첫 트랜잭션이 커밋(또는 롤백)할 때까지 `findByIdForUpdate`에서 대기한다. 이 대기 자체가 "정원을 넘기지 못하게 막는" 메커니즘이며, 재시도 로직이 필요 없다.

### 4-B. 낙관적 락으로 구현할 경우

`Study.java`에 버전 컬럼을 추가한다. 기존 필드·생성자·메서드는 그대로 두고 필드만 더한다.

```java
package co.dingcodingco.studypass.study;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Version;
import java.time.LocalDateTime;

@Entity
public class Study {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // ... title, category, fee, capacity, enrolledCount, status, openedAt, createdAt은 기존 그대로 ...

    @Version
    private Long version;

    // ... 기존 생성자·isFull()·enroll()·getter는 그대로 ...
}
```

`schema.sql`의 `study` 테이블 정의에 컬럼을 추가한다.

```sql
ALTER TABLE study ADD COLUMN version BIGINT NOT NULL DEFAULT 0;
```

`schema.sql` 자체를 고쳐도 이미 떠 있는 DB에는 반영되지 않으므로(3주차 인덱스 작업 때와 동일), 개발 DB에는 위 `ALTER TABLE`을 직접 실행하거나 `docker compose down -v && up -d`로 재기동한다.

`EnrollmentService`에 재시도 전용 메서드를 분리한다. 재시도는 **새 트랜잭션마다 다시 시도**해야 하므로, 실제 신청 로직(`@Transactional`)과 재시도 루프(트랜잭션 없음)를 서로 다른 메서드로 나눈다.

```java
package co.dingcodingco.studypass.enrollment;

import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.study.Study;
import co.dingcodingco.studypass.study.StudyRepository;
import java.time.LocalDateTime;
import org.springframework.orm.ObjectOptimisticLockingFailureException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class EnrollmentService {

    private static final int MAX_RETRY = 5;

    private final StudyRepository studyRepository;
    private final MemberRepository memberRepository;
    private final EnrollmentRepository enrollmentRepository;
    private final MemberEnrollmentStatsRepository memberEnrollmentStatsRepository;

    public EnrollmentService(
            StudyRepository studyRepository,
            MemberRepository memberRepository,
            EnrollmentRepository enrollmentRepository,
            MemberEnrollmentStatsRepository memberEnrollmentStatsRepository) {
        this.studyRepository = studyRepository;
        this.memberRepository = memberRepository;
        this.enrollmentRepository = enrollmentRepository;
        this.memberEnrollmentStatsRepository = memberEnrollmentStatsRepository;
    }

    /**
     * 4주차 미션: @Version 충돌 시 새 트랜잭션으로 재시도한다.
     * 트랜잭션이 없는 이 메서드가 재시도 루프를 돌고, 매 시도마다 doEnroll()이 새 트랜잭션을 연다.
     */
    public Long enroll(Long studyId, Long memberId) {
        for (int attempt = 1; attempt <= MAX_RETRY; attempt++) {
            try {
                return doEnroll(studyId, memberId);
            } catch (ObjectOptimisticLockingFailureException e) {
                if (attempt == MAX_RETRY) {
                    throw e;
                }
            }
        }
        throw new IllegalStateException("도달할 수 없는 경로");
    }

    @Transactional
    Long doEnroll(Long studyId, Long memberId) {
        Study study = studyRepository.findById(studyId)
                .orElseThrow(() -> new IllegalArgumentException("스터디를 찾을 수 없습니다: " + studyId));
        Member member = memberRepository.findById(memberId)
                .orElseThrow(() -> new IllegalArgumentException("회원을 찾을 수 없습니다: " + memberId));

        study.enroll();

        LocalDateTime enrolledAt = LocalDateTime.now();
        Enrollment enrollment =
                new Enrollment(study, member, "CONFIRMED", study.getFee(), enrolledAt);
        Enrollment saved = enrollmentRepository.save(enrollment);

        memberEnrollmentStatsRepository.increment(
                member.getId(), study.getFee(), enrolledAt
        );

        return saved.getId();
    }
}
```

`enroll()`과 `doEnroll()`이 같은 클래스 안에 있으면 스프링 프록시가 `doEnroll()`의 `@Transactional`을 가로채지 못해 실제로 새 트랜잭션이 열리지 않는다(self-invocation 문제). 검증 단계에서 로그로 커밋 시점을 확인하거나, 문제가 있으면 `doEnroll()`을 별도 `@Service`(예: `EnrollmentWriter`)로 분리한다.

### 선택 근거 기록 방법

어느 쪽을 선택하든 2단계 테스트 결과(개선 전/후)와 함께 다음을 evidence에 남긴다.

- 정합성: 개선 후 `enrolledCount`가 capacity를 넘지 않음
- 성공 수: 동시 요청 중 실제로 성공한 신청 수 (capacity와 일치해야 함)
- 재시도 수: 낙관적 락이면 재시도 횟수 합계, 비관적 락이면 "재시도 없음(대기로 처리)"을 재시도 수 항목에 명시

## 5단계: 트랜잭션 범위 점검

`EnrollmentService.enroll()` 안에는 이메일 발송이나 외부 API 호출 같은 트랜잭션 밖으로 뺄 작업이 현재 없다. 이 점검 결과 자체가 제출 증거이므로, "점검했지만 뺄 것이 없었다"고 정직하게 기록한다.

비관적 락을 선택한 경우, 락을 잡는 시점(`study` 조회)부터 커밋까지의 구간을 최소화하는 것이 트랜잭션 범위 조정의 핵심이며, 4-A 코드에서 이미 회원 조회를 `findByIdForUpdate` **이전**으로 옮겨 반영했다 — 회원 조회는 정원 경합과 무관하므로 락 밖에서 먼저 끝내면 락 보유 시간이 줄어든다.

- [ ] 트랜잭션 안 외부 호출 유무 점검 결과 기록
- [ ] 락 보유 구간을 줄일 수 있는지 점검하고 실제로 옮겼다면 그 근거 기록

## 6단계: 개선 후 재현 테스트로 검증

2단계의 `concurrentEnrollCanExceedCapacityBeforeFix()`를 그대로 복사해 이름과 검증 방식만 바꾼 테스트를 같은 파일에 추가한다. **capacity, 스레드 수, 동시 출발 방식은 그대로 두고** `assert`로 불변식을 강제하는 점만 다르다.

```java
@DisplayName("동시 요청 " + THREAD_COUNT + "건이 몰려도 락 적용 후에는 성공 수가 정확히 capacity로 맞춰진다 (개선 후)")
@RepeatedTest(5)
void concurrentEnrollRespectsCapacityAfterFix() throws InterruptedException {
    Study study = saveStudy();
    List<Long> memberIds = saveMembers(THREAD_COUNT);

    ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
    CountDownLatch ready = new CountDownLatch(THREAD_COUNT);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(THREAD_COUNT);
    AtomicInteger successCount = new AtomicInteger();

    for (Long memberId : memberIds) {
        executor.submit(() -> {
            ready.countDown();
            try {
                start.await();
                if (enroll(study.getId(), memberId).is2xxSuccessful()) {
                    successCount.incrementAndGet();
                }
            } catch (Exception ignored) {
            } finally {
                done.countDown();
            }
        });
    }

    ready.await();
    start.countDown();
    done.await(30, TimeUnit.SECONDS);
    executor.shutdown();

    Study reloaded = studyRepository.findById(study.getId()).orElseThrow();

    assertThat(reloaded.getEnrolledCount()).isEqualTo(CAPACITY);
    assertThat(successCount.get()).isEqualTo(CAPACITY);
}
```

`@RepeatedTest(5)`를 쓰려면 `org.junit.jupiter.api.RepeatedTest` import가 필요하다. 반복마다 `saveStudy()`로 새 study를 만들기 때문에 반복 간 간섭은 없다.

낙관적 락(4-B)을 선택했다면 재시도 때문에 응답이 실패로 끝나는 요청이 없어야 정상이다 — `MAX_RETRY`를 다 써도 충돌하면 그 요청은 예외를 던지고 실패로 집계되므로, `successCount`가 `CAPACITY`에 못 미치면 `MAX_RETRY`를 늘리거나 재시도 간 짧은 backoff를 추가해야 한다.

```bash
./gradlew test --tests '*EnrollmentConcurrencyTest*'
```

- [ ] 단일 스레드 통과 테스트(개선 후에도 그대로 통과)
- [ ] 동시 요청 테스트에서 불변식 유지 확인 (`@RepeatedTest(5)` 결과 모두 통과)

## 7단계: resume·evidence·PR

`resume/resume.md`에 실제 측정값이 나온 뒤 문장을 채운다. 형식 예시:

```text
스터디 신청 API에 [스레드 수]개 동시 요청을 [반복 횟수]회 재현해
정원 [capacity]명인 스터디에서 [개선 전 성공 수]건까지 신청이 받아지는 것을 확인한 뒤,
[선택한 락 전략]을 적용해 동시 요청에서도 성공 수를 정확히 capacity([N])건으로 맞췄다.
```

`evidence/week-04__weekly-pr.md`에는 다음을 빠짐없이 남긴다.

- [ ] capacity, 스레드 수, 요청 수, 반복 횟수 등 재현 조건
- [ ] 개선 전 정원 초과 재현 출력(성공 수 vs capacity)
- [ ] 선택한 락 전략과 구현 코드 위치
- [ ] 개선 후 같은 조건 재현 출력(불변식 유지, 반복 결과)
- [ ] 정합성·성공 수·재시도 수 세 가지 근거
- [ ] 트랜잭션 범위 점검 결과
- [ ] 근거형 질문 1~4의 최초 판단 → 연결한 코드/로그 → 검증 후 답변
- [ ] `## 리뷰 반영`: 최초에는 `자동 리뷰 수신 전`, 이후 지적 유무에 따라 갱신

제출 계약 확인:

- [ ] `resume/resume.md`, `src/main/`, `src/test/`, `evidence/week-04__weekly-pr.md`를 모두 변경했다.
- [ ] `missions/`, `challenge.json`, 채점 workflow는 변경하지 않았다.
- [ ] 개선 전/후 재현 테스트 출력이 저장소 안에 실제로 있다.

---

## 선택 확장 (필수 아님)

미션 5번 선택 확장: 두 번째 전략(네임드 락 등)을 같은 조건에서 비교하거나 데드락을 의도적으로 재현해 해결. 필수 항목을 먼저 통과시킨 뒤에만 고려한다.
