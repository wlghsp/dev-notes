# Week 3 실험 힌트

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
Part A(prep-questions.md) 개념 정리 후, JpaObservation으로 실제로 관찰할 실험들.

이름은 예시다. 의미가 통하면 바꿔도 된다. 정답 코드는 안 적는다 — 각 테스트가 "무엇을 준비하고, 무엇을 실행하고, 무엇을 관찰해야 하는지"만 구체화한다.

---

## 실험 전에: 엔티티 준비 상태 점검

기존 `User`(1)↔`Todo`(N, `@ManyToOne`/`@OneToMany`, `mappedBy = "user"`) 양방향 관계와 `Todo.setUser()` 편의 메서드가 이미 있다. B-1, B-2는 이 기존 관계로 바로 실험 가능하다.

- B-1, B-2: 추가 작업 없이 `User`/`Todo`로 진행 가능
- B-3(LAZY/N+1): `Todo.user`가 지금 `FetchType` 명시가 없어 `@ManyToOne` 기본값(EAGER)으로 동작한다. LAZY 초기화 시점을 관찰하려면 `@ManyToOne(fetch = FetchType.LAZY)`로 명시해야 실험이 성립한다 — 이건 기존 엔티티 수정으로 충분하고 새 엔티티는 필요 없다
- B-4(cascade/orphanRemoval): 지금 `User.todos`에는 `cascade`, `orphanRemoval` 옵션이 전혀 없다. 두 옵션을 켠 상태와 끈 상태를 대칭으로 비교해야 하므로, 기존 `User`/`Todo`에 옵션을 얹는 것만으로는 "켰을 때 vs 껐을 때" 대조가 안 된다 — 아래에서 별도 엔티티 쌍이 필요한 이유

### 테스트 클래스 구성 (추천)

패키지: `co.dingcodingco.challenge.week3` (기존 week1/week2 패키지 컨벤션과 동일)

- B-1 → `AssociationOwnershipTest` — `owner_side_field_change_reflects_in_db`, `non_owner_side_field_change_does_not_reflect_in_db`
- B-2 → `AssociationConvenienceMethodTest` — `without_convenience_method_object_graph_is_inconsistent`, `with_convenience_method_object_graph_is_consistent`
- B-3 → `LazyProxyAndNPlusOneTest` — `lazy_association_no_sql_before_access`, `lazy_association_sql_on_first_access`, `lazy_proxy_class_differs_from_entity_class`, `iterating_parents_and_accessing_lazy_children_causes_n_plus_one`
- B-4 → `CascadeAndOrphanRemovalTest` — `cascade_remove_deletes_children_with_parent`, `removing_child_from_collection_without_orphan_removal_keeps_child`, `removing_child_from_collection_with_orphan_removal_deletes_child`, `cascade_persist_saves_children_added_to_collection`(선택)

B-1/B-2를 하나로 합치지 않고 클래스를 나누는 이유: DB 관점(B-1)과 메모리 객체 그래프 관점(B-2)이 서로 다른 걸 검증하므로, concepts.md/prep-questions.md의 구분과 테스트 클래스 구조를 일치시키기 위함. B-4는 새로 만들 `Board`/`Comment` 엔티티를 쓰므로 `Todo`/`User` 기반 테스트(B-1~B-3)와 자연스럽게 분리된다.

---

## B-1. 연관관계 주인 확인

### `owner_side_field_change_reflects_in_db`

- DisplayName: "연관관계 주인 쪽 필드를 변경하면 flush 시 SQL이 반영된다"
- given: 양방향 연관관계를 가진 두 엔티티를 각각 저장해서 영속 상태로 만든다 (필요하면 `observation.reset()`으로 given 단계의 SQL을 관찰 대상에서 제외)
- when: 연관관계의 주인 쪽 필드(외래 키를 가진 쪽)를 변경하고 `flush()` 호출
- then: `observation.statements()`에 UPDATE(또는 INSERT) SQL이 실제로 포함되어 있는지 확인. SQL 문자열에 변경한 외래 키 컬럼명이 들어있는지까지 보면 더 확실

### `non_owner_side_field_change_does_not_reflect_in_db`

- DisplayName: "연관관계 주인이 아닌(mappedBy) 쪽 필드만 변경하면 flush해도 SQL이 나가지 않는다"
- given: B-1과 동일한 given (엔티티 두 개, 양방향 연관관계, 영속 상태)
- when: `mappedBy`가 붙은 반대쪽 필드(컬렉션 또는 참조)만 변경하고 `flush()` 호출
- then: `observation.statements()`가 비어있거나, 최소한 이 변경과 관련된 UPDATE/INSERT가 없는지 확인
- 대조 포인트: given은 거의 동일하되 "어느 쪽을 바꾸는가"만 다르다는 걸 두 테스트 이름과 구조에서 명확히 드러낼 것

---

## B-2. 편의 메서드 없이 vs 있이 — 객체 그래프 불일치 재현

### `without_convenience_method_object_graph_is_inconsistent`

- DisplayName: "편의 메서드 없이 주인 쪽만 설정하면 반대쪽 객체 그래프는 갱신되지 않는다"
- given: 두 엔티티를 저장해서 영속 상태로 만든다
- when: 편의 메서드를 쓰지 않고 주인 쪽 필드만 직접 세팅 (`flush()`는 아직 호출하지 않거나, 호출해도 무방 — 핵심은 "같은 영속성 컨텍스트 안"이라는 조건)
- then: 반대쪽 엔티티의 컬렉션(또는 참조)을 조회했을 때, 방금 추가한 엔티티가 비어있거나 안 보이는지 확인. 이 결과가 "실패해야 정상"이라는 걸 테스트 이름과 주석으로 명시
- 왜 필요한가: DB에는 문제가 없는데 객체 그래프만 깨진 상태라는 걸 증명하는 게 이 테스트의 목적. evidence에 "이 테스트가 왜 이렇게 나와야 정상인지" 이유를 적어야 함(association-ownership-and-cascade.md의 SQL vs 메모리 구분 참고)

### `with_convenience_method_object_graph_is_consistent`

- DisplayName: "편의 메서드로 주인 쪽을 설정하면 반대쪽 객체 그래프도 즉시 일관된다"
- given: B-2 첫 번째 테스트와 동일
- when: 편의 메서드로 주인 쪽을 세팅 (편의 메서드 내부에서 반대쪽 컬렉션도 함께 갱신)
- then: 반대쪽에서 즉시 조회해도 방금 추가한 엔티티가 보이는지 확인
- 대조 포인트: 이 두 테스트는 DB 쿼리 유무가 아니라 "메모리상 객체 그래프의 일관성"을 검증한다는 게 B-1과의 차이. `flush()` 여부와 무관하게 결과가 갈려야 함

---

## B-3. LAZY 프록시 초기화 시점

### `lazy_association_no_sql_before_access`

- DisplayName: "LAZY 연관관계는 접근 전까지 초기화 SQL이 나가지 않는다"
- given: LAZY로 선언된 연관관계를 가진 부모 엔티티를 저장하고, 새로운 트랜잭션/영속성 컨텍스트에서 부모를 조회
- when: 조회 직후 `observation.reset()` 호출 (부모 조회 시점의 SQL을 관찰 대상에서 제외하기 위함), 그리고 LAZY 필드에는 아직 접근하지 않음
- then: `observation.statements()`가 비어있는지 확인 — "조회했다"와 "초기화됐다"가 다르다는 걸 증명

### `lazy_association_sql_on_first_access`

- DisplayName: "LAZY 필드에 처음 접근하는 순간 초기화 SELECT SQL이 발생한다"
- given/when: 위 테스트와 동일하게 준비하고 reset까지 마침
- when(추가): LAZY 필드의 실제 값에 접근하는 메서드(단순 참조 대입이 아니라 getter 호출 등) 실행
- then: 그 시점에 `observation.statements()`에 SELECT SQL이 정확히 추가되었는지 확인. 접근 전/후로 나눠서 "접근이 트리거"라는 인과관계를 보여줘야 함

### `lazy_proxy_class_differs_from_entity_class`

- DisplayName: "초기화 전 LAZY 프록시의 클래스는 원본 엔티티 클래스와 다르다"
- given: LAZY 연관관계를 가진 부모 엔티티 조회 (초기화는 아직 안 시킴)
- when: 연관 필드 자체(참조)를 꺼내서 `getClass()` 호출 — 필드 값 접근이 아니라 참조 자체의 클래스를 보는 것이므로 초기화를 안 일으키는지도 같이 확인
- then: 반환된 클래스가 원본 엔티티 클래스와 다른지(`isNotEqualTo(Entity.class)`) 확인. 클래스 이름에 Hibernate 프록시임을 나타내는 접미사가 붙는지도 로그로 참고

### `iterating_parents_and_accessing_lazy_children_causes_n_plus_one`

"부모/자식"은 도메인상 1:N 관계(User가 부모, Todo가 자식)를 뜻하는 것이지 연관관계 주인과는 다른 축이다 — 주인은 여전히 Todo(`user_id` FK를 가진 쪽)다. 이 실험에서 "순회 대상"은 Todo 쪽이다: Todo 여러 건을 조회하고, 각 Todo의 LAZY `user`에 접근하는 방식으로 N+1을 재현한다.

- DisplayName: "Todo 목록을 순회하며 LAZY user에 접근하면 N+1 쿼리가 발생한다"
- given: `Todo`를 N개(예: 3개) 저장하고, 각각 다른 `User`(LAZY `@ManyToOne`)를 갖도록 준비
- when: `observation.reset()` 후, `Todo` 목록을 한 번에 조회(쿼리 1번 예상) → 그 목록을 순회하며 각 `Todo`의 `user`에 실제로 접근(예: `getUsername()`)
- then: `observation.statements()`의 크기가 "Todo 목록 조회 1번 + 순회 중 user 초기화 N번"과 일치하는지 확인. N값을 given에서 명시적으로 고정해야 "1 + N" 공식이 숫자로 검증됨

---

## B-4. cascade와 orphanRemoval 삭제 범위

### 엔티티 준비

orphanRemoval "있음 vs 없음"을 대칭으로 비교해야 하므로, `User`/`Todo`를 그대로 고치기보다 새 부모-자식 쌍이 필요하다. 방식은 두 가지 중 선택:

- (A) 새 엔티티 쌍 하나: 부모 엔티티(예: `Board`)와 자식 엔티티(예: `Comment`, `@ManyToOne`+`@OneToMany(mappedBy=..., cascade=CascadeType.REMOVE, orphanRemoval=true)`)를 만들고, "orphanRemoval 없음" 케이스는 같은 필드에서 옵션만 잠깐 끄고 돌리는 대신 별도 테스트 픽스처(예: 다른 필드 또는 다른 부모 클래스)로 분리
- (B) 부모 엔티티 하나에 자식 컬렉션 필드를 두 개 만들기: 예를 들어 `Board`에 `commentsWithOrphanRemoval`(orphanRemoval=true)과 `commentsWithoutOrphanRemoval`(orphanRemoval=false) 두 개 컬렉션 필드를 두면, 부모/자식 클래스는 하나씩만 만들고 매핑 옵션 차이로 대조군을 구성할 수 있음 — 다만 이 경우 자식 엔티티의 FK 컬럼을 어느 쪽 관계로 매핑할지 설계가 필요

(B)가 엔티티 클래스 수는 적지만 매핑이 다소 억지스러울 수 있어, 보통은 (A)처럼 새 부모/자식 엔티티 쌍을 하나 만들고 cascade/orphanRemoval을 실험 목적에 맞게 설정하는 편이 명확하다. cascade PERSIST 실험(`cascade_persist_saves_children_added_to_collection`)도 이 새 엔티티 쌍으로 함께 진행 가능 — `User`/`Todo`는 건드리지 않고 유지

### `cascade_remove_deletes_children_with_parent`

- DisplayName: "cascade REMOVE가 설정되어 있으면 부모 삭제 시 자식도 함께 삭제된다"
- given: cascade REMOVE가 설정된 연관관계로 부모 1개, 자식 1개 이상을 저장
- when: 부모 엔티티에 `repository.delete()`(또는 `remove()`) 호출, flush까지 진행
- then: 자식이 실제로 DB에서 삭제됐는지 확인(자식 repository로 재조회했을 때 없어야 함). `observation.statements()`에 자식 테이블 대상 DELETE가 포함되는지도 함께 확인 가능

### `removing_child_from_collection_without_orphan_removal_keeps_child`

- DisplayName: "orphanRemoval이 없으면 컬렉션에서 자식을 제거해도 DB에는 그대로 남는다"
- given: orphanRemoval이 **없는** 연관관계로 부모 1개, 자식 1개 이상을 저장 (부모는 삭제하지 않음)
- when: 부모의 컬렉션에서 자식을 `remove()`로만 제거하고 flush
- then: 자식이 DB에 그대로 남아있는지 확인 (또는 외래 키만 null로 바뀌었는지 — 매핑 방식에 따라 다름). 부모는 그대로 살아있다는 것도 함께 확인해서 "부모 삭제와 무관하다"는 전제를 보여줌

### `removing_child_from_collection_with_orphan_removal_deletes_child`

- DisplayName: "orphanRemoval이 있으면 컬렉션에서 자식을 제거하는 것만으로 DB에서 삭제된다"
- given: B-4 두 번째 테스트와 동일한 구조이되, orphanRemoval이 **있는** 엔티티(또는 필드)로 준비 — 대칭을 위해 가능하면 엔티티 구조 자체를 두 번째 테스트와 동일하게 맞추고 orphanRemoval 여부만 다르게
- when: 동일하게 컬렉션에서 자식을 `remove()`로만 제거하고 flush
- then: 자식이 DB에서 실제로 삭제됐는지 확인
- 대조 포인트: 이 테스트와 바로 위 테스트가 "같은 조작, 다른 설정, 다른 결과"라는 대칭을 이뤄야 orphanRemoval의 효과가 증명됨. 두 테스트를 나란히 놓고 evidence에서 비교

### `cascade_persist_saves_children_added_to_collection` (선택)

- DisplayName: "cascade PERSIST가 없으면 컬렉션에 추가만 한 자식은 부모 저장 시 함께 저장되지 않는다"
- given: cascade PERSIST가 없는 연관관계로 부모(아직 미저장, new 상태)를 준비하고 자식을 컬렉션에 추가만 함
- when: 부모만 `save()`(또는 `persist()`)
- then: 자식이 저장됐는지 확인 — prep-questions.md A-2에서 정리했듯 예외가 아니라 "조용히 무시"되는지 실제로 재현. cascade PERSIST가 있는 버전과 대조해서 두 번째 케이스도 만들면 더 명확
