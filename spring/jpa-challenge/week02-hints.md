# Week 2 진행 가이드 — 힌트만

레포: challenge-jpa-deep-dive-2026-08-wlghsp-r8
미션: 식별자와 save 함정 재현 및 근거형 답변

정답 없음. Week 1 도구(JpaObservation)를 재사용해서 "무엇을 관찰할지"만 짚는다.

- Part A. 개념 학습
- Part B. 실험 힌트
- Part C. 테스트 작성

---

## Part A. 개념 학습

### A-1. 근거형 질문 직결 개념

- 엔티티 생명주기 4단계 중 "준영속"이 정확히 어떤 상태인가?
  - 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 것을 준영속 상태라고한다.
  - 영속성 컨텍스트가 제공하는 기능을 사용하지 못함
  - 준영속 상태 만드는 법
    - em.detach(entity) — 특정 엔티티만 준영속 전환, 1차 캐시에서 빠짐
    - em.clear() — 컨텍스트 완전 초기화, 테스트에서 주로 사용
    - em.close() — 영속성 컨텍스트 종료
- persist()/merge()는 각각 어떤 상태를 인자로 받는가? 잘못 넣으면?
  - persist()는 New/Transient만, merge()는 Detached만 대상
  - 잘못 넣으면 EntityExistsException, OptimisticLockException 등 발생하거나 데이터가 올바르게 반영 안 됨
- IDENTITY/SEQUENCE는 식별자를 언제 확보하는가?
  - IDENTITY는 INSERT 후, SEQUENCE는 INSERT 전
- save()는 persist/merge를 무엇으로 판단하는가?
  - 식별자 상태 + Persistable 구현 여부. 판단은 isNew()가 담당(SimpleJpaRepository / EntityInformation)
  - isNew(): 원시타입 id는 0이면 신규, 래퍼타입 id는 null이면 신규
- flush/commit 차이, `@DataJpaTest`의 트랜잭션 처리는?
  - flush: 쓰기 지연 SQL을 DB로 전송. 트랜잭션은 아직 안 끝남. 1차 캐시 유지
  - commit: flush 후 트랜잭션 종료, 변경을 영구 반영
  - `@DataJpaTest`: 메서드마다 트랜잭션 시작, 종료 시 자동 롤백. 필요하면 `@Rollback(false)`로 방지 가능

이 5가지는 근거형 질문 1~4와 대응한다.

### A-2. 실험 중 부딪히는 것들

- 준영속 객체 setter는 변경 감지가 걸리는가? 그 주체는 관리 범위 밖 객체도 추적하는가?
- merge() 외에 준영속을 영속으로 되돌리는 방법이 있는가? 병합 시점에 필드가 null이면?
- equals/hashCode 미재정의 상태에서 "같다"의 기준은? 그 기준이 성립하는 범위는?
- 영속성 컨텍스트와 `@Transactional` 범위의 관계는? 트랜잭션 종료 시 그 안의 영속 객체는?
- 어떤 객체가 영속 상태인지 코드로 물어볼 방법은?
- merge()는 연관 엔티티까지 전이시키는가? 그걸 결정하는 설정은?
- `@Id` 직접 할당 시 save()의 신규 판단 기준은 그대로 통하는가?
- SEQUENCE의 `allocationSize`는 무엇을 하는가? 매 persist마다 채번 SQL이 안 나가면 그 사이 id는 어디서 오는가?

---

## Part B. 실험 힌트

이름은 예시다. 의미가 통하면 바꿔도 된다.

### B-1. persist vs merge 경로 비교

- `persist_new_entity` — persist() 직후 SQL이 안 나가는지, id는 있는지
- `merge_detached_entity` — merge() 반환 객체와 인자로 넘긴 객체가 같은 객체인지(`isSameAs`)
- `persist_on_already_persisted_entity_throws` (선택) — 이미 영속인 엔티티를 다시 persist()하면 무슨 일이 나는지
- `merge_with_nonexistent_id_inserts_as_new` (선택) — 존재하지 않는 id로 merge()하면 무슨 일이 나는지, 그 id가 실제로 저장되는지

### B-2. INSERT 시점이 전략마다 다르다

비교는 조건이 최소 두 개 나란히 있어야 성립한다. `Todo`(AUTO/SEQUENCE) 하나만으로는 "다르다"를 증명 못 한다.

준비: `Todo`는 그대로 두고 `@GeneratedValue(strategy = GenerationType.IDENTITY)`만 다른 엔티티를 하나 추가한다(필드는 id, title이면 충분). 자동 스캔 범위 안이면 설정 없이 persist 가능.

- `persist_with_sequence_id_is_ready_before_insert` — 기존 `persistTest`를 이 이름으로 볼 수도 있음: id는 있는데 INSERT는 아직
- `persist_with_identity_inserts_immediately` — IDENTITY 엔티티로 persist() 직후 id 상태와 INSERT 여부 확인. 위와 결과가 반대로 나와야 "다르다"가 증명됨
- `persist_with_assigned_id` (선택, 미션 문구의 "직접 할당") — `@GeneratedValue` 없이 `@Id`만 있는 엔티티로 persist 시점 확인

두 테스트(또는 세 테스트)가 정확히 대칭되는 given/when 구조를 갖게 만들고, 결과 차이를 evidence에서 1차 캐시(identity map)가 식별자를 키로 쓴다는 사실과 연결해 설명한다.

### B-3. merge 반환 객체 혼동 회귀 테스트

- `using_detached_reference_after_merge_does_not_reflect_change` — 전달한 detached 객체를 계속 들고 변경 → flush 후에도 반영 안 됨을 재현 (실패해야 정상인 케이스)
- `using_returned_entity_after_merge_reflects_change` — 반환값으로 변경 → flush 후 반영됨 (대조군)
- 두 테스트는 given이 거의 동일하고 when 이후만 갈린다

### B-4. flush와 commit 구분

- `commit_persists_data_beyond_flush` (가칭) — `@DataJpaTest`의 롤백을 막는 스프링 기능을 찾아 적용. 같은 테스트 메서드가 여전히 하나의 트랜잭션 안이라는 제약과 부딪힐 것 — commit이 flush와 다르다는 걸 그 안에서 증명 가능한지 직접 확인
- 증명이 애매하다면, 왜 애매한지 자체가 근거형 질문 4번 답의 재료가 될 수 있다
- 롤백을 안 하면 데이터가 남는다 — cleanup을 같은 테스트에 넣을지 별도로 뺄지는 판단해서 결정

---

## Part C. 테스트 작성

- `observation.reset()`은 관찰하려는 동작 직전에 호출 — given 단계 SQL과 섞이지 않게
- given에 "준영속 객체 만들기" 절차가 새로 추가됨 (B-1 참고). 길어지면 헬퍼로 분리
- SQL 필터링은 `startsWith`로 충분한 것도 있지만, IDENTITY vs SEQUENCE처럼 순서 자체를 봐야 하면 `statements()` 리스트 인덱스를 직접 비교
- 동일성/혼동 검증은 `isSameAs`/`isNotSameAs`
- 같은 시나리오를 전략별로 반복하면 테스트 이름에 전략을 명시 (`insert_timing_with_identity` 등)
- 회귀 테스트(B-3)는 "왜 존재하는지"를 evidence에 설명

---

## 막히면

- 질문 1은 `SimpleJpaRepository`를 직접 열어봐라. 식별자 존재 여부만이 아니라 `Persistable`, `@Version` 유무도 갈린다.
- IDENTITY와 쓰기 지연이 왜 상충하는지는, 쓰기 지연 SQL 저장소가 식별자를 왜 미리 알아야 하는지 거꾸로 생각해봐라.
