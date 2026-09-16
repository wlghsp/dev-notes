# 3주차 회귀 테스트 구현 코드

8단계에서 추가할 테스트는 복합 인덱스나 집계 테이블의 내부 구현이 아니라, 개선 뒤에도 API 결과가 이전 계약과 같은지를 검증한다.

- `search`: 기간·status·최소 금액 조건, from/to 포함 경계, 최신순 정렬
- `stats`: `minFee` 이상 구간만의 회원별 count·totalFee·lastEnrolledAt

## 사전 조건

테스트는 `src/test/resources/application.yml`의 H2 DB에서 실행된다. 따라서 `src/main/resources/schema.sql`에도 집계 테이블 DDL이 있어야 한다. 로컬 MySQL에만 수동으로 테이블을 만들면 테스트에서는 `Table "MEMBER_ENROLLMENT_FEE_STATS" not found`가 난다.

```sql
create table if not exists member_enrollment_fee_stats (
    member_id bigint not null,
    fee int not null,
    enrollment_count bigint not null,
    total_fee bigint not null,
    last_enrolled_at datetime(6) not null,
    primary key (member_id, fee),
    index idx_member_enrollment_fee_stats_fee_member (fee, member_id),
    constraint fk_member_enrollment_fee_stats_member
        foreign key (member_id) references member (id)
);
```

이 테스트는 stats 조회 결과를 검증하는 목적이므로, 테스트 준비 단계에서는 `JdbcTemplate`으로 집계 테이블에 알려진 값을 넣는다. MySQL 전용 UPSERT의 문법 호환성을 H2 테스트가 대신 검증하려는 것이 아니다. 신규 신청 시 집계를 갱신하는 UPSERT는 실제 MySQL에서 별도로 확인한다.

## `EnrollmentControllerTest` 전체 코드

새 파일 경로:

```text
src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentControllerTest.java
```

```java
package co.dingcodingco.studypass.enrollment;

import static org.assertj.core.api.Assertions.assertThat;

import co.dingcodingco.studypass.member.Member;
import co.dingcodingco.studypass.member.MemberRepository;
import co.dingcodingco.studypass.study.Study;
import co.dingcodingco.studypass.study.StudyRepository;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;
import java.util.UUID;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.jdbc.core.JdbcTemplate;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class EnrollmentControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private EnrollmentRepository enrollmentRepository;

    @Autowired
    private MemberRepository memberRepository;

    @Autowired
    private StudyRepository studyRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    private Member commonMember;
    private Study commonStudy;

    @BeforeEach
    void setUp() {
        String suffix = UUID.randomUUID().toString();
        commonMember = memberRepository.save(
                new Member("fixture-" + suffix + "@studypass.test", "공통 테스트 회원"));
        commonStudy = studyRepository.save(
                new Study("공통 테스트 스터디", "BACKEND", 50_000, 10,
                        LocalDateTime.of(2026, 7, 1, 0, 0)));
    }

    @DisplayName("신청 검색은 기간·상태·최소 금액을 적용하고 최신 신청부터 반환한다")
    @Test
    void searchFiltersByConditionsAndOrdersByNewestFirst() {
        LocalDateTime from = LocalDateTime.of(2026, 8, 1, 0, 0);
        LocalDateTime middle = LocalDateTime.of(2026, 8, 15, 12, 0);
        LocalDateTime to = LocalDateTime.of(2026, 8, 31, 23, 59, 59);
        String targetStatus = "SEARCH_TEST";

        Enrollment atFrom = saveEnrollment(commonMember, targetStatus, 50_000, from);
        Enrollment atMiddle = saveEnrollment(commonMember, targetStatus, 70_000, middle);
        Enrollment atTo = saveEnrollment(commonMember, targetStatus, 60_000, to);

        saveEnrollment(commonMember, targetStatus, 49_999, middle);
        saveEnrollment(commonMember, "OTHER_STATUS", 70_000, middle);
        saveEnrollment(commonMember, targetStatus, 70_000, from.minusSeconds(1));

        ResponseEntity<Map<String, Object>> response = restTemplate.exchange(
                "/api/enrollments/search"
                        + "?from=2026-08-01T00:00:00"
                        + "&to=2026-08-31T23:59:59"
                        + "&status=SEARCH_TEST"
                        + "&minFee=50000",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<>() {});

        assertThat(response.getStatusCode().is2xxSuccessful()).isTrue();
        assertThat(response.getBody()).isNotNull();
        assertThat(((Number) response.getBody().get("count")).longValue()).isEqualTo(3L);

        List<?> content = (List<?>) response.getBody().get("content");
        assertThat(content)
                .allSatisfy(row -> assertThat(row).isInstanceOf(Map.class))
                .extracting(row -> ((Number) ((Map<?, ?>) row).get("id")).longValue())
                .containsExactly(atTo.getId(), atMiddle.getId(), atFrom.getId());
    }

    @DisplayName("회원별 신청 통계는 최소 금액 이상 신청만 회원 단위로 합산한다")
    @Test
    void statsAggregatesOnlyEnrollmentsAtOrAboveMinimumFee() {
        Member firstMember = commonMember;
        Member secondMember = saveMember("통계 회원 2");

        // 원본 데이터: 기존 stats 쿼리가 계산하던 근거 데이터다.
        saveEnrollment(firstMember, "CONFIRMED", 20_000,
                LocalDateTime.of(2026, 8, 1, 10, 0));
        saveEnrollment(firstMember, "CONFIRMED", 50_000,
                LocalDateTime.of(2026, 8, 10, 10, 0));
        saveEnrollment(firstMember, "CONFIRMED", 70_000,
                LocalDateTime.of(2026, 8, 20, 12, 0));
        saveEnrollment(secondMember, "CONFIRMED", 80_000,
                LocalDateTime.of(2026, 8, 15, 10, 0));
        saveEnrollment(secondMember, "CONFIRMED", 80_000,
                LocalDateTime.of(2026, 8, 25, 10, 0));

        // 집계 테이블은 위 원본을 회원·금액별로 미리 계산한 값이다.
        insertStats(firstMember.getId(), 20_000, 1, 20_000,
                LocalDateTime.of(2026, 8, 1, 10, 0));
        insertStats(firstMember.getId(), 50_000, 1, 50_000,
                LocalDateTime.of(2026, 8, 10, 10, 0));
        insertStats(firstMember.getId(), 70_000, 1, 70_000,
                LocalDateTime.of(2026, 8, 20, 12, 0));
        insertStats(secondMember.getId(), 80_000, 2, 160_000,
                LocalDateTime.of(2026, 8, 25, 10, 0));

        ResponseEntity<List<Map<String, Object>>> response = restTemplate.exchange(
                "/api/enrollments/stats?minFee=50000",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<>() {});

        assertThat(response.getStatusCode().is2xxSuccessful()).isTrue();
        assertThat(response.getBody()).isNotNull();

        Map<String, Object> firstStats = findStats(response.getBody(), firstMember.getId());
        assertThat(((Number) firstStats.get("enrollmentCount")).longValue()).isEqualTo(2L);
        assertThat(((Number) firstStats.get("totalFee")).longValue()).isEqualTo(120_000L);
        assertThat(firstStats.get("lastEnrolledAt")).isEqualTo("2026-08-20T12:00");

        Map<String, Object> secondStats = findStats(response.getBody(), secondMember.getId());
        assertThat(((Number) secondStats.get("enrollmentCount")).longValue()).isEqualTo(2L);
        assertThat(((Number) secondStats.get("totalFee")).longValue()).isEqualTo(160_000L);
        assertThat(secondStats.get("lastEnrolledAt")).isEqualTo("2026-08-25T10:00");
    }

    private void insertStats(
            Long memberId,
            int fee,
            long enrollmentCount,
            long totalFee,
            LocalDateTime lastEnrolledAt) {
        jdbcTemplate.update(
                """
                INSERT INTO member_enrollment_fee_stats
                    (member_id, fee, enrollment_count, total_fee, last_enrolled_at)
                VALUES (?, ?, ?, ?, ?)
                """,
                memberId,
                fee,
                enrollmentCount,
                totalFee,
                lastEnrolledAt);
    }

    private Member saveMember(String nickname) {
        String suffix = UUID.randomUUID().toString();
        return memberRepository.save(
                new Member("fixture-" + suffix + "@studypass.test", nickname));
    }

    private Enrollment saveEnrollment(
            Member member, String status, int fee, LocalDateTime enrolledAt) {
        return enrollmentRepository.save(
                new Enrollment(commonStudy, member, status, fee, enrolledAt));
    }

    private Map<String, Object> findStats(List<Map<String, Object>> stats, Long memberId) {
        return stats.stream()
                .filter(row -> ((Number) row.get("memberId")).longValue() == memberId)
                .findFirst()
                .orElseThrow(() -> new AssertionError("통계에 회원이 없습니다: " + memberId));
    }
}
```

## 실행과 해석

```bash
./gradlew test --tests '*EnrollmentControllerTest'
./gradlew test
```

두 테스트가 검증하는 것은 다음과 같다.

| 테스트 | 고정하는 API 계약 |
| --- | --- |
| `searchFiltersByConditionsAndOrdersByNewestFirst` | from/to와 `fee == minFee`를 포함하고, 다른 status·낮은 금액·기간 밖 신청을 제외하며 최신순으로 반환한다. |
| `statsAggregatesOnlyEnrollmentsAtOrAboveMinimumFee` | `fee == minFee`를 포함해 회원별 count·금액 합계·가장 최근 신청 시각을 반환한다. |

인덱스 사용 여부는 이 테스트가 아니라 동일 조건의 EXPLAIN으로 확인한다. 테스트는 인덱스를 바꾸거나 집계 조회 구현을 리팩터링해도 API 결과가 달라지지 않도록 보호하는 역할이다.
