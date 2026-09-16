# stats 집계 테이블 구현 코드

`GET /api/enrollments/stats`가 원본 `study_enrollment` 20만 건 대신 `member_enrollment_fee_stats`를 읽도록 바꾸는 구현 예시다.

## 1. 조회 결과 구현체 추가

새 파일:

```text
src/main/java/co/dingcodingco/studypass/enrollment/MemberEnrollmentStatsRow.java
```

```java
package co.dingcodingco.studypass.enrollment;

import java.time.LocalDateTime;

public record MemberEnrollmentStatsRow(
        Long memberId,
        long enrollmentCount,
        long totalFee,
        LocalDateTime lastEnrolledAt
) implements MemberEnrollmentStats {

    @Override
    public Long getMemberId() {
        return memberId;
    }

    @Override
    public long getEnrollmentCount() {
        return enrollmentCount;
    }

    @Override
    public long getTotalFee() {
        return totalFee;
    }

    @Override
    public LocalDateTime getLastEnrolledAt() {
        return lastEnrolledAt;
    }
}
```

기존 `MemberEnrollmentStats` 인터페이스를 그대로 구현하므로 Controller의 응답 변환 코드를 재사용할 수 있다.

## 2. 집계 테이블 repository 추가

새 파일:

```text
src/main/java/co/dingcodingco/studypass/enrollment/MemberEnrollmentStatsRepository.java
```

```java
package co.dingcodingco.studypass.enrollment;

import java.time.LocalDateTime;
import java.util.List;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

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
                (rs, rowNum) -> new MemberEnrollmentStatsRow(
                        rs.getLong("member_id"),
                        rs.getLong("enrollment_count"),
                        rs.getLong("total_fee"),
                        rs.getTimestamp("last_enrolled_at").toLocalDateTime()),
                minFee);
    }

    public void increment(Long memberId, int fee, LocalDateTime enrolledAt) {
        jdbcTemplate.update(
                """
                INSERT INTO member_enrollment_fee_stats
                    (member_id, fee, enrollment_count, total_fee, last_enrolled_at)
                VALUES (?, ?, 1, ?, ?) AS incoming
                ON DUPLICATE KEY UPDATE
                    enrollment_count = member_enrollment_fee_stats.enrollment_count
                        + incoming.enrollment_count,
                    total_fee = member_enrollment_fee_stats.total_fee
                        + incoming.total_fee,
                    last_enrolled_at = GREATEST(
                        member_enrollment_fee_stats.last_enrolled_at,
                        incoming.last_enrolled_at)
                """,
                memberId,
                fee,
                fee,
                enrolledAt);
    }
}
```

조회 시 fee 구간별 통계를 다시 회원 단위로 합산한다. 원본 쿼리의 `COUNT`, `SUM(fee)`, `MAX(enrolled_at)` 의미를 그대로 유지한다.

`increment()`는 `(member_id, fee)` 행이 없으면 생성하고, 있으면 기존 count와 total을 증가시킨다. `VALUES(column)` 함수 대신 삽입 행 별칭 `incoming`을 사용하므로 MySQL 8.0.20 이후의 폐기 예정 경고가 발생하지 않는다.

## 3. Controller 조회 변경

`EnrollmentController`에 repository 필드와 생성자 인자를 추가한다.

```java
private final MemberEnrollmentStatsRepository memberEnrollmentStatsRepository;

public EnrollmentController(
        EnrollmentRepository enrollmentRepository,
        EnrollmentService enrollmentService,
        MemberEnrollmentStatsRepository memberEnrollmentStatsRepository) {
    this.enrollmentRepository = enrollmentRepository;
    this.enrollmentService = enrollmentService;
    this.memberEnrollmentStatsRepository = memberEnrollmentStatsRepository;
}
```

`stats()`의 조회 시작 부분을 변경한다.

```java
@GetMapping("/stats")
public List<Map<String, Object>> stats(@RequestParam(defaultValue = "0") int minFee) {
    return memberEnrollmentStatsRepository.findByMinFee(minFee).stream()
            .map(row -> Map.<String, Object>of(
                    "memberId", row.getMemberId(),
                    "enrollmentCount", row.getEnrollmentCount(),
                    "totalFee", row.getTotalFee(),
                    "lastEnrolledAt", String.valueOf(row.getLastEnrolledAt())))
            .toList();
}
```

SQL에 `LIMIT 50`이 있으므로 기존 stream의 `.limit(50)`은 제거한다.

기존 `EnrollmentRepository.statsByMember()`는 더 이상 사용되지 않으므로 삭제한다. 남겨두면 실제 API가 어느 쿼리를 사용하는지 혼동하기 쉽다.

## 4. 신규 신청 시 집계 갱신

`EnrollmentService`에 repository 필드와 생성자 인자를 추가한다.

```java
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
```

`enroll()`의 마지막 부분을 다음처럼 변경한다.

```java
LocalDateTime enrolledAt = LocalDateTime.now();
Enrollment enrollment =
        new Enrollment(study, member, "CONFIRMED", study.getFee(), enrolledAt);
Enrollment saved = enrollmentRepository.save(enrollment);

memberEnrollmentStatsRepository.increment(
        member.getId(), study.getFee(), enrolledAt);

return saved.getId();
```

`enroll()`에 이미 `@Transactional`이 있으므로 신청 INSERT와 집계 UPSERT가 같은 트랜잭션에 참여한다. 어느 한쪽에서 예외가 발생하면 둘 다 롤백된다.

## 5. SeedRunner 백필 연결

`SeedRunner` 안에 백필 메서드를 추가한다.

```java
private void backfillEnrollmentStats() {
    jdbcTemplate.update("""
            INSERT INTO member_enrollment_fee_stats
                (member_id, fee, enrollment_count, total_fee, last_enrolled_at)
            SELECT
                aggregated.member_id,
                aggregated.fee,
                aggregated.new_enrollment_count,
                aggregated.new_total_fee,
                aggregated.new_last_enrolled_at
            FROM (
                SELECT
                    member_id,
                    fee,
                    COUNT(*) AS new_enrollment_count,
                    SUM(fee) AS new_total_fee,
                    MAX(enrolled_at) AS new_last_enrolled_at
                FROM study_enrollment
                GROUP BY member_id, fee
            ) AS aggregated
            ON DUPLICATE KEY UPDATE
                enrollment_count = aggregated.new_enrollment_count,
                total_fee = aggregated.new_total_fee,
                last_enrolled_at = aggregated.new_last_enrolled_at
            """);
}
```

기존 신청 데이터가 있을 때의 조기 반환을 다음처럼 바꾼다. 신청은 존재하지만 집계 테이블이 비어 있을 때만 최초 백필한다.

```java
Long existing = jdbcTemplate.queryForObject(
        "select count(*) from study_enrollment", Long.class);

Long existingStats = jdbcTemplate.queryForObject(
        "select count(*) from member_enrollment_fee_stats", Long.class);

if (existing != null && existing > 0) {
    if (existingStats == null || existingStats == 0) {
        backfillEnrollmentStats();
    }
    return;
}
```

새 시드를 모두 만든 뒤에도 호출한다. `study_enrollment` 배치 INSERT와 마지막 `study.enrolled_count` 갱신이 끝난 다음에 두면 된다.

```java
backfillEnrollmentStats();
```

동작 조건은 다음과 같다.

| 신청 데이터 | 집계 데이터 | 동작 |
| --- | --- | --- |
| 없음 | 없음 | 시드 생성 후 백필 |
| 있음 | 없음 | 기존 신청을 최초 백필 |
| 있음 | 있음 | 백필 없이 바로 반환 |

백필 이후의 신규 신청은 `EnrollmentService.enroll()`에서 집계 테이블도 같은 트랜잭션으로 갱신한다. 따라서 정상 상태에서는 앱을 재시작할 때 전체 백필을 반복할 필요가 없다.

## 6. 최소 확인

```bash
./gradlew test

curl -sS -G 'http://localhost:8080/api/enrollments/stats' \
  --data-urlencode 'minFee=50000' | jq '.[0:3]'
```

API 결과가 기존 원본 집계와 같은지 확인한 다음, 새 집계 테이블을 대상으로 EXPLAIN과 5회 응답 시간을 다시 측정한다.
