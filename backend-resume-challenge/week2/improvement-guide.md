# 2주차 개선 적용 가이드: 커서 기반 페이징 전환

execution-guide.md 5단계(개선 적용)의 실제 코드 변경 가이드. 1주차에 예측한 두 병목(count 쿼리 풀스캔, OFFSET 페이징) 중 OFFSET 페이징을 커서 기반으로 전환한다.

## 왜 커서 페이징을 선택했는가

- `study` 테이블은 `id`가 PK(자동 인덱스)라서 별도 인덱스 설계 없이 바로 `WHERE id > ?` 조건을 인덱스로 탈 수 있음
- 기존 회귀 테스트(1주차 `StudyControllerTest`)의 응답 형태(`content`)를 크게 안 바꾸고 개선 가능
- count 문제(Slice 전환)는 API 응답 계약 자체를 더 크게 바꾸는 결정이라 이번 주차 범위를 넘어섬 — 이건 별도 개선 후보로 남겨둠

트레이드오프: 임의 페이지로 점프하는 기능(`page=5`)과 총 개수(`totalElements`)를 포기하고, "다음 페이지"만 가능한 방식으로 바뀜.

---

## 1단계: `StudyRepository`에 커서 조회 메서드 추가

```java
package co.dingcodingco.studypass.study;

import java.util.List;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudyRepository extends JpaRepository<Study, Long> {

    List<Study> findByCategory(String category);

    List<Study> findTop10ByStatusOrderByEnrolledCountDesc(String status);

    // 추가: 커서(id) 이후의 데이터를 id 오름차순으로 size만큼 조회
    List<Study> findByIdGreaterThanOrderByIdAsc(Long cursor, Pageable pageable);
}
```

`Pageable`은 여기서 정렬용이 아니라 개수 제한(LIMIT)만 쓰는 용도 (`PageRequest.of(0, size)` 형태로 넘김).

## 2단계: `StudyController.list()`를 커서 기반으로 변경

```java
@GetMapping
public Map<String, Object> list(
        @RequestParam(required = false) Long cursor,
        @RequestParam(defaultValue = "20") int size) {
    int limitedSize = Math.min(size, 100);
    long baseCursor = (cursor != null) ? cursor : 0L;

    List<Study> studies = studyRepository.findByIdGreaterThanOrderByIdAsc(
            baseCursor, PageRequest.of(0, limitedSize));

    Long nextCursor = studies.isEmpty() ? null : studies.get(studies.size() - 1).getId();

    return Map.of(
            "nextCursor", nextCursor,
            "content", studies.stream().map(StudyController::toSummary).toList());
}
```

- `page` 파라미터와 `totalElements`가 응답에서 사라짐 (커서 페이징의 근본적인 트레이드오프)
- `import org.springframework.data.domain.PageRequest;`는 유지, `Page<Study>` import는 이제 안 씀

## 3단계: 기존 회귀 테스트 수정

기존 `listReturnsOkWithExpectedFields`가 `totalElements`를 검증하고 있다면 `nextCursor`로 교체:

```java
@Test
void listReturnsOkWithExpectedFields() {
    studyRepository.save(new Study("테스트 스터디", "BACKEND", 10000, 5, LocalDateTime.now()));

    ResponseEntity<Map> response = restTemplate.getForEntity("/api/studies?size=20", Map.class);

    assertThat(response.getStatusCode().is2xxSuccessful()).isTrue();
    assertThat(response.getBody()).containsKeys("nextCursor", "content");

    List<Map<String, Object>> content = (List<Map<String, Object>>) response.getBody().get("content");
    assertThat(content).isNotEmpty();
    assertThat(content.get(0)).containsKeys(
            "id", "title", "category", "fee", "capacity", "enrolledCount");
}
```

`sizeParameterIsCappedAt100`은 `cursor` 없이 `size=500`만 보내는 식으로 그대로 재사용 가능(응답 필드만 `content.size()` 확인이라 로직 안 바뀜).

### 새 테스트 추가: 커서 동작 자체 검증

```java
@Test
void cursorReturnsNextPageAfterGivenId() {
    Study s1 = studyRepository.save(new Study("A", "BACKEND", 10000, 5, LocalDateTime.now()));
    Study s2 = studyRepository.save(new Study("B", "BACKEND", 10000, 5, LocalDateTime.now()));

    ResponseEntity<Map> response = restTemplate.getForEntity(
            "/api/studies?cursor=" + s1.getId() + "&size=20", Map.class);

    List<Map<String, Object>> content = (List<Map<String, Object>>) response.getBody().get("content");
    assertThat(content).extracting(m -> m.get("id"))
            .doesNotContain(s1.getId().intValue())
            .contains(s2.getId().intValue());
}
```

## 4단계: 실행 후 SQL 로그 확인

`logging.level.org.hibernate.SQL: debug`가 이미 켜져 있으니, 서버 재시작 후 확인:

```bash
curl "localhost:8080/api/studies?cursor=0&size=20"
```

로그에서 확인할 것:
- `WHERE id > ? ORDER BY id ASC LIMIT ?` 형태로 나가는지
- `count(*)` 쿼리가 더 이상 없는지 (원래 count는 그대로 남아있는 게 정상 — OFFSET 문제만 해결했으므로 count 쿼리 자체는 이번 개선 범위 밖. 다만 `list()`가 더 이상 `Page`를 안 쓰므로 count 쿼리도 이번엔 함께 사라짐)

이 로그가 evidence의 "개선 코드와 선택한 기법의 근거"에 들어갈 핵심 증거.

## 5단계: k6 재측정

execution-guide.md 6단계로 이동해서 baseline.js를 다시 실행. 다만 baseline.js가 `page=0&size=20`으로 고정되어 있다면, 커서 파라미터에 맞게 스크립트도 함께 수정이 필요할 수 있음 — `k6-scripts/baseline.js`의 요청 URL을 `?cursor=&size=20` 형태로 맞춰야 개선 후 API를 정확히 때림.
