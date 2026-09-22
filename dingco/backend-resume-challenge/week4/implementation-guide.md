# 4주차 구현 가이드

`week4/prep-questions.md`는 아직 답변이 비어 있다. 이 문서는 그 답을 전제로 하지 않고, `missions/README.md` Week 4 요구사항과 `studypass`(`/Users/jihochoi/Documents/study/dingco/challenge-backend-resume-2026-08-wlghsp-r17`) 현재 코드를 근거로 "무엇을 어떤 순서로 구현할지"만 정리한다. 실행 결과·수치·판단 근거는 실제로 실행한 뒤 evidence와 prep-questions에 채운다.

---

## 0. 현재 코드 상태 확인

경합 지점은 이미 두 곳에 주석으로 표시되어 있다.

- `Study.enroll()` (`Study.java`): `isFull()`로 `enrolledCount >= capacity`를 확인한 뒤 `enrolledCount + 1`. check와 act 사이에 원자성이 없다.
- `EnrollmentService.enroll()` (`EnrollmentService.java`): `@Transactional`이지만 study 조회 → `study.enroll()` → Enrollment 저장 → 집계 갱신 순서로만 되어 있고 락이나 원자적 갱신은 없다.

`EnrollmentController.enroll()`은 `POST /api/enrollments/studies/{studyId}?memberId=`로 이미 노출되어 있으므로 컨트롤러 변경은 필요 없다.

## 0-1. 선행 수정: `MemberEnrollmentStatsRepository.increment()`가 테스트 환경(H2)에서 깨짐

2단계 테스트를 실제로 돌려본 결과 드러난 기존 버그다. `src/test/resources/application.yml`은 테스트를 MySQL 없이 H2(`MODE=MySQL`)로 돌리도록 되어 있는데, `MemberEnrollmentStatsRepository.increment()`(`MemberEnrollmentStatsRepository.java`)가 MySQL 8 전용 `INSERT ... VALUES (...) AS incoming ON DUPLICATE KEY UPDATE ...` 문법을 쓴다. H2는 이 문법을 파싱하지 못해 `enroll()`을 호출하는 모든 요청이 500으로 죽는다(`BadSqlGrammarException`). 3주차 테스트는 `enroll()`을 거치지 않고 리포지토리에 직접 데이터를 넣는 방식이라 이 문제가 지금까지 드러나지 않았다.

`increment()`를 UPDATE 시도 후 영향받은 행이 없으면 INSERT하는 방식으로 바꿔 MySQL과 H2 양쪽에서 동일하게 동작하도록 고쳤다.

```java
package co.dingcodingco.studypass.enrollment;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;

@Repository
public class MemberEnrollmentStatsRepository {
    private final JdbcTemplate jdbcTemplate;

    public MemberEnrollmentStatsRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<MemberEnrollmentStats> findByMinFee(int minFee) {
        return jdbcTemplate.query(
                """
                    SELECT member_id,
                           SUM(enrollment_count) AS enrollment_count,
                           SUM(total_fee) AS total_fee,
                           MAX(last_enrolled_at) AS last_enrolled_at
                    FROM member_enrollment_fee_stats
                    WHERE fee >= ?
                    GROUP BY member_id
                    ORDER BY enrollment_count DESC
                    LIMIT 50
                    """,
                (rs, row) -> new MemberEnrollmentStatsRow(
                        rs.getLong("member_id"),
                        rs.getLong("enrollment_count"),
                        rs.getLong("total_fee"),
                        rs.getTimestamp("last_enrolled_at").toLocalDateTime()
                ), minFee
        );
    }

    /**
     * MySQL 전용 INSERT ... ON DUPLICATE KEY UPDATE 대신, PK(member_id, fee) 존재 여부에
     * 따라 UPDATE 또는 INSERT를 선택하는 방식으로 바꿨다. MySQL과 테스트용 H2(MODE=MySQL)
     * 양쪽에서 동일하게 동작한다.
     */
    public void increment(Long memberId, int fee, LocalDateTime enrolledAt) {
        int updated = jdbcTemplate.update(
                """
                UPDATE member_enrollment_fee_stats
                SET enrollment_count = enrollment_count + 1,
                    total_fee = total_fee + ?,
                    last_enrolled_at = CASE
                        WHEN last_enrolled_at >= ? THEN last_enrolled_at
                        ELSE ?
                    END
                WHERE member_id = ? AND fee = ?
                """,
                fee, enrolledAt, enrolledAt, memberId, fee);

        if (updated == 0) {
            jdbcTemplate.update(
                    """
                    INSERT INTO member_enrollment_fee_stats
                        (member_id, fee, enrollment_count, total_fee, last_enrolled_at)
                    VALUES (?, ?, 1, ?, ?)
                    """,
                    memberId, fee, fee, enrolledAt);
        }
    }
}
```

주의할 점: UPDATE-then-INSERT 방식은 같은 `(memberId, fee)` 조합으로 두 트랜잭션이 동시에 처음 들어오면, 둘 다 UPDATE에서 0건(아직 행이 없어서)을 보고 둘 다 INSERT를 시도해 PK 중복 에러가 날 수 있다. 4주차 동시성 테스트는 매번 서로 다른 회원을 새로 만들어 같은 `(memberId, fee)`가 동시에 부딪히지 않으므로 지금 재현 테스트에는 영향이 없지만, 실제 운영에서 같은 회원이 같은 fee로 정확히 동시에 신청하는 경우까지 고려한다면 이 메서드도 별도 동시성 보완이 필요하다는 점은 evidence에 한계로 남긴다.

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

    @Autowired
    private EnrollmentRepository enrollmentRepository;

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
        long savedEnrollmentCount = enrollmentRepository.countByStudyId(study.getId());

        // 개선 전에는 assert로 강제하지 않고, 실제로 넘긴 값을 그대로 기록해 evidence에 옮긴다.
        System.out.printf(
                "[개선 전] successCount=%d, savedEnrollmentCount=%d, enrolledCount=%d, capacity=%d%n",
                successCount.get(), savedEnrollmentCount, reloaded.getEnrolledCount(), CAPACITY);
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

**순서 주의**: 아래 4-A/4-B는 `EnrollmentService.enroll()`을 그 자리에서 락 버전으로 교체한다. 즉 교체하고 나면 저장소에는 "개선 전 코드"가 더 이상 남아 있지 않다(3주차 search/stats 개선 때도 같은 방식 — `EnrollmentRepository`의 원래 쿼리를 인덱스 적용 후 쿼리로 그대로 바꿨다). 그래서 **2단계에서 만든 `concurrentEnrollCanExceedCapacityBeforeFix()`를 반드시 이 교체 전에 먼저 실행해서 결과(성공 수·savedEnrollmentCount·enrolledCount)를 기록해둔다.** 교체 후에는 같은 테스트를 다시 돌려도 이미 락이 걸려 있어 "개선 전" 상황이 재현되지 않는다.

- [ ] `enroll()` 교체 전, `concurrentEnrollCanExceedCapacityBeforeFix()` 실행 결과를 evidence용으로 별도 기록
- [ ] 기록 후에만 아래 4-A 또는 4-B로 `EnrollmentService.enroll()`을 교체

미션 필수 요건은 낙관적/비관적 중 **하나만** 선택하는 것이지만, 선택 확장 5번("두 번째 전략을 같은 조건에서 비교")이 이미 "둘 다 구현해서 비교"를 예상한 길이다. 이 가이드는 둘 다 구현하는 순서로 안내한다.

**구조**: `EnrollmentService.enroll()`은 최종 채택한 전략만 남긴다(운영 코드는 하나, 기존 `POST /api/enrollments/studies/{studyId}` 경로가 그대로 이걸 탄다). 비교용으로 만드는 낙관적 락은 별도 클래스 `EnrollmentOptimisticFacade`로 분리하고, 이를 실제로 호출하는 **비교 전용 엔드포인트**(예: `POST /api/enrollments/studies/{studyId}/optimistic`)를 `EnrollmentController`에 추가한다. 이렇게 하면 기존 `EnrollmentConcurrencyTest`가 쓰는 `TestRestTemplate` + HTTP 호출 방식을 그대로 재사용해 두 전략을 같은 조건으로 비교할 수 있다. 두 구현이 최종 PR에 함께 남으므로, "실제 서비스가 쓰는 경로는 어느 쪽인지"를 evidence에 명시해야 리뷰에서 "왜 두 구현이 다 있나"라는 지적을 방어할 수 있다.

권장 순서:

1. 4-A(비관적 락)를 `EnrollmentService.enroll()`에 먼저 적용하고, 6단계 검증까지 마쳐 개선 후 수치를 확보한다.
2. `EnrollmentService.enroll()`은 그대로 둔 채, 4-B(낙관적 락)를 별도 클래스 `EnrollmentOptimisticFacade`로 새로 만든다(아래 4-B 코드의 클래스명을 `EnrollmentService` 대신 `EnrollmentOptimisticFacade`로 바꾸고, 같은 필드·생성자 구조를 그대로 쓴다).
3. `EnrollmentController`에 `EnrollmentOptimisticFacade`를 호출하는 비교 전용 엔드포인트를 추가한다.

   ```java
   @PostMapping("/studies/{studyId}/optimistic")
   public Map<String, Object> enrollOptimistic(
           @PathVariable Long studyId, @RequestParam Long memberId) {
       Long id = enrollmentOptimisticFacade.enroll(studyId, memberId);
       return Map.of("enrollmentId", id);
   }
   ```

   `EnrollmentController` 생성자에 `EnrollmentOptimisticFacade` 의존성을 추가로 주입받는다.
4. `EnrollmentConcurrencyTest`에 이 엔드포인트(`/optimistic`)를 호출하는 동일 조건(capacity·스레드 수·`CountDownLatch` 구조) 테스트를 추가해 재시도 수까지 포함해 측정한다. 기존 `enroll()` 헬퍼 메서드를 복사해 경로만 `/optimistic`으로 바꾼 버전을 쓰면 된다.
5. 두 결과(정합성·성공 수·재시도 수)를 evidence-draft.md의 "선택 근거" 표로 비교하고, 이 프로젝트 상황(짧은 트랜잭션, 순간적으로 몰리는 요청)에 비춰 `EnrollmentService.enroll()`에 최종 남길 전략을 확정한다. 이 프로젝트 상황상 비관적 락이 기본 후보다 — 경합이 몰리는 짧은 트랜잭션은 재시도 로직 없이 대기만으로 정합성을 보장하기 쉽다.

필수 항목만 먼저 통과시키고 싶다면 1번까지만 하고 넘어가도 된다. 아래 4-A/4-B는 각 전략의 기준 코드이며, 비교 구조로 갈 때는 4-B를 위 2~3번처럼 별도 클래스·별도 엔드포인트로 재배치한다.

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

비교 구조로 간다면(위 "권장 순서" 참고) 아래 `EnrollmentService`는 `EnrollmentOptimisticFacade`라는 새 클래스로 만든다 — 기존 `EnrollmentService.enroll()`(비관적 락, 4-A)은 그대로 두고 이 클래스를 추가하는 것이다. 필수 항목만 하고 낙관적 락 하나만 최종 채택한다면 이 클래스명을 `EnrollmentService`로 그대로 쓰고 기존 파일을 교체하면 된다.

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

재시도는 **새 트랜잭션마다 다시 시도**해야 한다. Spring의 `@Transactional`은 프록시 기반으로 동작하는데, 같은 클래스 안에서 `this.doEnroll(...)`처럼 자기 자신을 호출하면 프록시를 거치지 않고 원본 객체로 직행해 `@Transactional`이 통째로 무시된다(self-invocation 문제). "검증하다 문제 있으면 분리"가 아니라, 이 문제를 피하려면 **처음부터** 재시도 루프와 실제 트랜잭션 로직을 별도 클래스로 나눠야 한다.

이 둘의 관계는 Facade 패턴이다. `EnrollmentController`와 `EnrollmentConcurrencyTest`는 `EnrollmentOptimisticService`의 존재나 "재시도가 몇 번 일어나는지"를 몰라도 되고, `EnrollmentOptimisticFacade.enroll(studyId, memberId)`라는 단일 창구만 호출한다. `EnrollmentOptimisticFacade`는 재시도라는 부가 관심사를 감싸는 파사드이고, `EnrollmentOptimisticService`가 트랜잭션 하나짜리 실제 작업(신청 저장)을 맡는다. 이 분리가 self-invocation 문제를 피하는 방법이면서 동시에, "재시도 정책"과 "신청 로직"이라는 서로 다른 책임을 구조적으로도 나누는 결과가 된다. `enroll()`에서 `service.doEnroll(...)`을 호출할 때 `service`가 스프링이 주입한 진짜 프록시 객체이므로 `@Transactional`이 정상 작동한다.

```java
package co.dingcodingco.studypass.enrollment;

import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.study.Study;
import co.dingcodingco.studypass.study.StudyRepository;
import java.time.LocalDateTime;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 4주차 미션: EnrollmentOptimisticFacade(재시도 루프)가 매 시도마다 호출하는
 * 실제 신청 트랜잭션이다. @Transactional이 붙은 이 메서드는 반드시 별도 빈을 거쳐
 * 프록시로 호출되어야 하므로, 재시도 루프와 같은 클래스에 두지 않는다(self-invocation 문제).
 */
@Service
public class EnrollmentOptimisticService {

    private final StudyRepository studyRepository;
    private final MemberRepository memberRepository;
    private final EnrollmentRepository enrollmentRepository;
    private final MemberEnrollmentStatsRepository memberEnrollmentStatsRepository;

    public EnrollmentOptimisticService(
            StudyRepository studyRepository,
            MemberRepository memberRepository,
            EnrollmentRepository enrollmentRepository,
            MemberEnrollmentStatsRepository memberEnrollmentStatsRepository) {
        this.studyRepository = studyRepository;
        this.memberRepository = memberRepository;
        this.enrollmentRepository = enrollmentRepository;
        this.memberEnrollmentStatsRepository = memberEnrollmentStatsRepository;
    }

    @Transactional
    public Long doEnroll(Long studyId, Long memberId) {
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

```java
package co.dingcodingco.studypass.enrollment;

import org.springframework.orm.ObjectOptimisticLockingFailureException;
import org.springframework.stereotype.Service;

/**
 * 4주차 미션: 비관적 락(EnrollmentService)과 같은 조건으로 비교하기 위한 낙관적 락 구현이다.
 * EnrollmentController의 /studies/{studyId}/optimistic 비교 전용 엔드포인트에서만 호출된다.
 * 실제 운영 경로(/studies/{studyId})는 EnrollmentService(비관적 락)를 그대로 쓴다.
 *
 * 이 클래스에는 @Transactional을 두지 않는다 — 매 재시도마다 EnrollmentOptimisticService의
 * 프록시를 거쳐 새 트랜잭션을 열어야 하기 때문이다(self-invocation 문제 회피).
 */
@Service
public class EnrollmentOptimisticFacade {

    private static final int MAX_RETRY = 5;

    private final EnrollmentOptimisticService service;

    public EnrollmentOptimisticFacade(EnrollmentOptimisticService service) {
        this.service = service;
    }

    public Long enroll(Long studyId, Long memberId) {
        for (int attempt = 1; attempt <= MAX_RETRY; attempt++) {
            try {
                return service.doEnroll(studyId, memberId);
            } catch (ObjectOptimisticLockingFailureException e) {
                if (attempt == MAX_RETRY) {
                    throw e;
                }
            }
        }
        throw new IllegalStateException("도달할 수 없는 경로");
    }
}
```

`EnrollmentController`에 `EnrollmentOptimisticFacade`를 주입받아 호출하는 비교 전용 엔드포인트를 추가한다(위 "권장 순서" 3번 코드와 동일). `EnrollmentOptimisticService`는 컨트롤러가 직접 부르지 않고 `EnrollmentOptimisticFacade`를 통해서만 호출된다.

```java
private final EnrollmentOptimisticFacade enrollmentOptimisticFacade;

// 생성자에 파라미터·대입 추가

@PostMapping("/studies/{studyId}/optimistic")
public Map<String, Object> enrollOptimistic(
        @PathVariable Long studyId, @RequestParam Long memberId) {
    Long id = enrollmentOptimisticFacade.enroll(studyId, memberId);
    return Map.of("enrollmentId", id);
}
```

필수 항목만 하고 낙관적 락 하나만 최종 채택하는 경우에도, 파사드(재시도 루프)와 트랜잭션 로직 분리 구조 자체는 유지해야 한다 — self-invocation 문제는 클래스를 하나만 쓰든 비교 구조로 가든 똑같이 발생하기 때문이다. 다만 이 경우 두 클래스명을 각각 `EnrollmentFacade`/`EnrollmentService`로 바꿔 기존 `EnrollmentService.java`를 대체하고, 비교 전용 엔드포인트(`/optimistic`)는 만들지 않는다.

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

4단계에서 `enroll()`을 이미 락 버전으로 교체했으므로, 이 시점부터 `concurrentEnrollCanExceedCapacityBeforeFix()`를 다시 돌려도 "개선 전" 결과가 아니라 "락이 걸린 상태"의 결과가 나온다. 그래서 개선 전 수치는 4단계 교체 직전에 이미 따로 기록해뒀어야 한다(못했다면 `git stash`로 4단계 변경을 잠시 되돌리고 재현 테스트를 다시 돌려 기록한 뒤 `git stash pop`으로 복원한다).

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
