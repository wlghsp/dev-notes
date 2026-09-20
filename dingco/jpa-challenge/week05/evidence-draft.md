# week-05 evidence 드래프트

다 채운 뒤 challenge-jpa-deep-dive-2026-08-wlghsp-r8/evidence/week-05__weekly-pr.md 로 옮긴다.

## 변경

- `Todo`를 `domain.todo` 패키지로 이동 (User와 depth 통일)
- `TodoSearchCondition`(record) — title, username 둘 다 optional
- `TodoSearchResult`(record, `@QueryProjection`) — todoId, title, username
- `TodoQueryRepository` — `search()`(개선 전, 엔티티 조회) / `searchWithProjection()`(개선 후, DTO projection)
- `TodoQueryController` — `GET /todos?title=&username=`
- `TodoQueryRepositoryTest` 5개(조건 조합, null 무시, N+1 재현, 개선 후 쿼리 1개, 결과 정합성)

## 검증

실행: `mvn test -Dtest=TodoQueryRepositoryTest` — 5개 전부 통과

fixture: User 2명(alice, bob), Todo 3건. 조회 조건 title="보고서" → 2건 매칭, 각각 다른 User.

**개선 전 — `search()`**

```sql
(여기에 로그 붙여넣기 — 목록 조회 1 + LAZY user 초기화 2 = 3개 예상)
```

**개선 후 — `searchWithProjection()`**

```sql
(여기에 로그 붙여넣기 — join 1개 쿼리 예상)
```

## 선택 근거

- fetch join 대신 DTO projection: 조회 전용 API라 변경 감지 불필요, 필요한 컬럼만 조회
- `Projections.constructor` 대신 `@QueryProjection`: 생성자 시그니처 안 맞으면 컴파일 타임에 잡힘
- `TodoSearchCondition`/`TodoSearchResult`는 record: 엔티티가 아닌 순수 DTO라 가능. `Todo`/`User` 등 엔티티는 record 불가(상속 불가, LAZY 프록시 생성 불가, 가변 id와 불일치)
- 정합성 = 결과 개수 + title 값 일치(정렬 후 비교)

## 근거형 질문

1. N+1은 LAZY/EAGER 중 어디서 발생하는가?
- 근거: `search_causes_n_plus_one_when_accessing_user` — 목록 조회 1 + LAZY user 초기화 2 = 3
- 답변: 양쪽 다 발생 가능. 원인은 연관 엔티티를 조인 없이 개별 SELECT로 가져오는 JPA 기본 동작 때문 — LAZY/EAGER는 그 시점만 다름

2. DTO projection이 이번 요구에 맞는 이유는?
- 근거: `search_with_projection_avoids_n_plus_one` — join 1개, SELECT 절에 필요한 컬럼만
- 답변: 목록 화면에 title/username만 필요하고 수정 계획 없어 영속 엔티티가 필요 없음

3. null·빈 값 처리 규칙은?
- 근거: `search_ignores_null_condition`
- 답변: `StringUtils.hasText()`로 null·공백 모두 걸러 `BooleanExpression`에 null 반환 → where절에서 자동 제외

4. 1차 캐시·fixture 순서 통제는?
- 근거: `createFixtures()`에서 `flush()` → `clear()`, 그 다음 `observation.reset()`
- 답변: 순서 안 지키면 fixture INSERT가 통계에 섞이거나, 1차 캐시에 남은 User 때문에 N+1이 재현 안 됨

## 리뷰 반영

자동 리뷰 수신 전
