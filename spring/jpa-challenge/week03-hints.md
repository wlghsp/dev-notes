# Week 3 진행 가이드 — 힌트만

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
미션: 연관관계와 삭제 경계 검증 및 근거형 답변

정답 없음. Week 1~2 도구(JpaObservation)를 재사용해서 "무엇을 관찰할지"만 짚는다.

- Part A. 개념 학습
- Part B. 실험 힌트
- Part C. 테스트 작성

---

## Part A. 개념 학습

### A-1. 근거형 질문 직결 개념

정답은 안 적는다. 각 질문에서 무엇을 코드로 직접 확인해야 답이 나오는지만 짚는다.

- 외래 키를 실제로 관리하는 연관관계 주인이 필요한 이유는 무엇인가?
  - 확인할 것: 양방향 매핑에서 `mappedBy`가 어느 쪽에 붙는지, 그 반대쪽 필드를 바꿨을 때 SQL이 나가는지 안 나가는지(B-1)

객체와 테이블의 외래 키 관리 기준을 일치시키고 데이터 무결성을 유지하기 위함입니다.
mappedBy는 주인이 아닌 쪽에 붙고, 변경을 해도 SQL은 나가지 않습니다. 

- LAZY로 선언해도 N+1이 발생할 수 있는 이유는 무엇인가?
  - 확인할 것: LAZY가 정확히 언제 SQL을 발생시키는지(초기화 트리거), 컬렉션을 순회하며 LAZY 필드를 하나씩 건드릴 때 그 트리거가 몇 번 반복되는지(B-3)

주엔티티를 조회할 때 1개의 쿼리가 실행. 이때 연관된 엔티티는 프록시(가짜 객체)상태로 가져옴. 조회한 주 엔티티 리스트를 순회하며 연관된 엔티티의 실제필드를 호출하면 JPA는 프록시를 실제 엔티티로 변환하기 위해 데이터베이스에 추가 쿼리를 날립니다. 
그 결과 부모 데이터 1개 조회를 위해 실행한 쿼리와 가져온 데이터 갯수(N)만큼 연관된 자식 데이터를 조회하는 쿼리가 더해져 N + 1 쿼리가 실행됩니다.

- cascade REMOVE와 orphanRemoval은 어떤 상황에서 결과가 달라지는가?
  - 확인할 것: 두 옵션이 각각 "무엇의 생명주기"를 추적하는지(부모의 삭제 vs 관계의 끊어짐), 부모는 안 지우고 자식만 컬렉션에서 제거했을 때 둘의 결과가 갈리는지(B-4)

JPA에서 1번 cascade= CascadeType.REMOVE와 2번 orphanRemoval = true는 둘 다 부모 엔티티를 삭제할 때 자식 엔티티가 함께 삭제되는 공통점이 있습니다. 

하지만 부모 엔티티와 자식 엔티티의 관계(연관관계)를 끊었을 때의 동작에서 결정적인 차이가 발생합니다. 
1번은 연관 관계 외래 키만 NULL이 되어 자식 에티티는 그대로 유지됨
2번은 고아 객체로 판단하여 DELETE 실행되어 자식 엔티티가 DB에서 삭제됨

- 연관관계 편의 메서드가 데이터베이스가 아니라 객체 상태를 위해 필요한 이유는 무엇인가?
  - 확인할 것: 주인 쪽만 설정하고 flush하기 전에 반대쪽에서 조회하면 무엇이 보이는지, 그게 SQL 문제인지 메모리 상의 객체 그래프 문제인지(B-2)

자바 객체 메모리 상에서 양방향 참조를 모두 동기화하여 객체의 일관성을 유지하기 위해 필요합니다.
메모리(1차 캐시)상의 데이터 불일치: 영속성 컨텍스트에 저장된 객체 상태가 완벽하지 않으면, DB에 반영(flush)되기 전 비즈니스 로직이나 단위 테스트를 실행할 때 문제가 생깁니다.

주인쪽만 설정하고 반대쪽에서 조회하면 해당 엔티티는 조회가 안되며, 그것은 SQL 문제가 아니라 메모리상의 객체 그래프 문제입니다.

이 4가지는 근거형 질문 1~4와 대응한다.

### A-2. 실험 중 부딪히는 것들

- 비주인 쪽 필드만 바꾸고 flush하면 SQL이 나가는가? 주인 쪽 필드는 그대로인 채로 반대쪽만 바꾸면 DB에는 뭐가 반영되는가?
- 프록시 객체의 클래스는 원본 엔티티 클래스와 같은가? `getClass()`로 비교하면 어떤 결과가 나오는가?
- LAZY 필드에 접근하지 않고 그냥 들고만 있으면 초기화가 안 일어나는가? `toString()`이나 로깅 코드가 실수로 LAZY를 건드릴 수 있는 지점은 어디인가?
- 준영속 상태의 프록시에 접근하면 어떤 예외가 나는가? (`LazyInitializationException`)
- cascade PERSIST 없이 자식을 컬렉션에 추가만 하고 부모를 저장하면 자식은 저장되는가?
- orphanRemoval을 컬렉션이 아니라 단일 `@OneToOne`에도 걸 수 있는가? 그때 "고아"는 어떤 상태를 뜻하는가?
- FetchType.LAZY인 `@ManyToOne`에 `@Fetch(FetchMode.JOIN)` 같은 걸 걸면 무슨 일이 나는가? (선택, 참고용)

---

## Part B. 실험 힌트

이름은 예시다. 의미가 통하면 바꿔도 된다.

### B-1. 연관관계 주인 확인

- `owner_side_field_change_reflects_in_db` — 주인 쪽 필드 변경 후 flush → 외래 키 UPDATE/INSERT SQL 확인
- `non_owner_side_field_change_does_not_reflect_in_db` — `mappedBy` 쪽만 변경 후 flush → SQL 안 나가는지 확인 (대조군)
- 두 테스트는 given이 거의 동일하고 "어느 쪽 필드를 바꾸는가"만 다르게 만든다

### B-2. 편의 메서드 없이 vs 있이 — 객체 그래프 불일치 재현

- `without_convenience_method_object_graph_is_inconsistent` — 주인 쪽만 설정 → 같은 영속성 컨텍스트 안에서 반대쪽 컬렉션 조회 시 비어있음을 확인 (실패해야 정상인 케이스, flush 전)
- `with_convenience_method_object_graph_is_consistent` — 편의 메서드로 양쪽 다 설정 → 반대쪽에서도 즉시 보임 (대조군)
- 이 두 테스트는 DB 상태가 아니라 영속성 컨텍스트 내 메모리 상태를 검증한다는 게 B-1과의 차이

### B-3. LAZY 프록시 초기화 시점

- `lazy_association_no_sql_before_access` — 부모 조회 직후 `observation.reset()` → LAZY 필드 접근 전까지 SQL 없음 확인
- `lazy_association_sql_on_first_access` — 위 상태에서 LAZY 필드의 실제 값(getter 등)에 접근 → 그 시점에 SELECT SQL 발생 확인
- `lazy_proxy_class_differs_from_entity_class` — 초기화 전 프록시의 `getClass()`와 실제 엔티티 클래스 비교 (Hibernate 프록시 확인)
- N+1 재현: `iterating_parents_and_accessing_lazy_children_causes_n_plus_one` — 부모 여러 건 조회 후 각각 LAZY 컬렉션 순회 → SQL 개수가 (부모 조회 1) + (순회 N)인지 `statements()` 크기로 확인

### B-4. cascade와 orphanRemoval 삭제 범위

- `cascade_remove_deletes_children_with_parent` — 부모 `remove()` → 연관된 자식도 함께 DELETE되는지 확인
- `removing_child_from_collection_without_orphan_removal_keeps_child` — orphanRemoval 없는 상태에서 자식을 컬렉션에서만 제거 → flush 후에도 자식이 DB에 남아있는지 확인
- `removing_child_from_collection_with_orphan_removal_deletes_child` — orphanRemoval 있는 엔티티로 동일 시나리오 → 자식이 DELETE되는지 확인. 위 테스트와 정확히 대칭 구조로 만들어야 "다르다"가 증명됨
- `cascade_persist_saves_children_added_to_collection` (선택) — 자식을 컬렉션에 추가만 하고 부모 저장 → cascade PERSIST 설정에 따라 자식 INSERT 여부가 갈리는지 확인

---

## Part C. 테스트 작성

- `observation.reset()`은 관찰하려는 동작 직전에 호출 — given 단계 SQL과 섞이지 않게
- LAZY 접근 여부를 확인할 땐 어떤 코드가 "접근"으로 카운트되는지 주의 — 단순 참조 보관과 필드/메서드 호출을 구분
- B-4의 두 대조 테스트(orphanRemoval 유무)는 엔티티를 따로 두거나, 설정이 다른 필드를 나란히 두는 방식 중 하나를 선택해서 대칭 구조를 유지
- 컬렉션 순회 시 SQL 개수를 셀 땐 `statements()` 리스트 크기를 given 이후 시점 기준으로 비교
- 동일성/불일치 검증은 컬렉션이 비어있는지(`isEmpty()`), 프록시 클래스 비교는 `isNotEqualTo(Entity.class)` 등으로
- B-2(양방향 불일치) 테스트는 "왜 실패해야 정상인지"를 evidence에 설명

---

## 막히면

- 질문 1은 매핑 애노테이션에서 `mappedBy`가 붙은 쪽이 어디인지부터 코드로 확인해라. `mappedBy`가 없는 쪽이 외래 키를 관리하는 쪽이다.
- 질문 2는 프록시가 "초기화됐다"는 게 정확히 무슨 뜻인지부터 짚어라 — 초기화는 어떤 트리거로 일어나는가, 그 트리거가 순회 코드 안에서 몇 번 발생하는가.
- 질문 3은 REMOVE와 orphanRemoval 각각의 "관찰 대상"이 다르다는 걸 기억해라 — 하나는 부모의 생명주기, 하나는 관계 자체의 생명주기.
