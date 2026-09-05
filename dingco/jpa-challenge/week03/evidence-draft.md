# Week 3 evidence 초안 (간결 버전)

studypass의 evidence/week-03__weekly-pr.md에 옮겨 적을 내용.

---

## 변경

1. `User`-`Todo`의 연관관계 주인을 `Todo.user`(`@ManyToOne`, FK 보유)로 정하고, `User.todos`는 `mappedBy`로 비주인 선언
`Todo.setUser()`를 편의 메서드로 만들어 주인 필드 설정 시 반대쪽 컬렉션도 함께 동기화 하도록 함. 대조 실험을 위해 편의 메서드를 거치지 않는 `setUserOnly()`도 별도로 추가
2. `Todo.user`에 `fetch = FetchType.LAZY`를 명시(기존에는 `@ManyToOne` 기본값인 EAGER였음). LAZY 초기화 시점과 N+1을 관찰하기 위함.
3. cascade와 orphanRemoval 삭제 범위 검증을 위해 `Board`/`Comment`(orphanRemoval 있음), `Post`/`Reply`(orphanRemoval 없음) 두 쌍의 엔티티를 새로 추가.
4. week3 패키지에 테스트 4개 클래스, 총 10개 테스트 작성:
  - `AssociationOwnershipTest` - 연관관계 주인 확인
  - `AssociationConvenienceMethodTest` - 편의 메서드 유무에 따른 객체 그래프 일관성
  - `LazyProxyAndPlusOneTest` - LAZY 초기화 시점과 N+1 재현
  - `CascadeAndOrphanRemovalTest` - cascade REMOVE/PERSIST와 orphanRemoval 삭제 범위


## 검증

- 실행: `mvn -Dtest=AssociationOwnershipTest,AssociationConvenienceMethodTest,LazyProxyAndNPlusOneTest,CascadeAndOrphanRemovalTest test`
- 결과: 10개 전부 통과
- 핵심 확인 사항:
  - 주인(`Todo.user`) 변경 시만 SQL 발생, 비주인(`User.todos`) 변경은 SQL 없음
  - 편의 메서드 없이 주인만 세팅 시 DB는 정상 반영되지만 `user.getTodos()`는 비어있음(메모리 불일치) — 편의 메서드 쓰면 둘 다 일치
  - LAZY는 접근 전 SQL 없음 → 접근 시점에 SQL 1개 발생. Todo 3건 순회 시 쿼리 1+3=4개(N+1 재현)
  - cascade REMOVE(부모 삭제 시 자식 삭제) / orphanRemoval(컬렉션 제거만으로 자식 삭제) 각각의 트리거를 대칭 테스트로 구분 확인
  - cascade PERSIST 있으면 컬렉션에 추가만 한 자식도 부모 저장 시 함께 INSERT됨

## 선택 근거

- `Board.comments`는 `CascadeType.ALL` 대신 `{PERSIST, REMOVE}`만 명시 — 이번 실험에 필요한 것만 최소로 선택.
- **실전에서 겪은 문제**: 처음엔 `cascade = REMOVE`(단독) + `orphanRemoval = true`로 했는데, 컬렉션에서 `remove()`해도 삭제가 전혀 트리거 안 됨. Hibernate 버전(6.2.13/6.4.10/6.5.3)을 낮춰가며 확인했지만 세 버전 모두 동일해 버전 문제 아님을 확인. `cascade`에 `PERSIST`를 추가하니 정상 동작. 이후 검색으로 이게 오래된 Hibernate 이슈(HHH-9571로 알려짐, 5.4.x 시절부터 보고됨)임을 확인해 저희 재현이 실제 존재하는 버그와 일치함을 재확인. 다만 그 소스 레벨 원인(왜 REMOVE 단독일 때 orphan 판정이 스킵되는지)까지는 못 밝혔고, 재현·해결책·버그 존재 확인까지만 근거로 남김.

## 근거형 질문

### 1. 외래 키를 실제로 관리하는 연관관계 주인이 필요한 이유는?
- 판단: 테이블 FK는 한쪽에만 있어 SQL 반영 기준이 필요
- 근거: `AssociationOwnershipTest` — 주인 변경 시만 SQL 발생, 비주인은 없음
- 답변: 주인의 상태 변경만 DB에 반영된다는 규칙을 로그로 확인함

### 2. LAZY로 선언해도 N+1이 발생할 수 있는 이유는?
- 판단: LAZY는 "안 가져온다"만 보장, "묶어서 가져온다"는 별개
- 근거: `LazyProxyAndNPlusOneTest` — 접근 전 SQL 없음, 접근 시 발생, Todo 3건 순회 시 1+3=4개
- 답변: 초기화 시점만 늦출 뿐 묶음 조회는 별도 장치(fetch join 등) 필요, 없으면 접근 횟수만큼 쿼리 반복됨을 확인

### 3. cascade REMOVE와 orphanRemoval은 어떤 상황에서 결과가 달라지는가?
- 판단: REMOVE는 부모 생명주기, orphanRemoval은 관계 생명주기 — 부모는 그대로, 컬렉션에서만 제거할 때 갈림
- 근거: `CascadeAndOrphanRemovalTest`의 대칭 테스트 2개
- 답변: 이론대로 확인. 추가로 `REMOVE` 단독 + `orphanRemoval`이 컬렉션 조작만으론 트리거 안 되는 실전 버그를 재현했고, `PERSIST`를 함께 둬야 정상 동작함을 확인함(Hibernate 버전 문제 아님도 확인)

### 4. 연관관계 편의 메서드가 데이터베이스가 아니라 객체 상태를 위해 필요한 이유는?
- 판단: 주인만 세팅해도 DB는 정상, 메모리 반대쪽 컬렉션만 안 갱신됨
- 근거: `AssociationConvenienceMethodTest` — 편의 메서드 없이는 DB 정상·`user.getTodos()` 비어있음, 편의 메서드 쓰면 둘 다 정상
- 답변: 같은 테스트에서 DB 반영과 메모리 상태를 같이 확인해 SQL 문제가 아니라 메모리 객체 그래프 문제임을 대조로 증명함

## 리뷰 반영

자동 리뷰 수신 전
